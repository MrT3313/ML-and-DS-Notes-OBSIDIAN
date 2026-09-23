---
note_kind: method
aliases:
  - imputation
  - impute
  - imputer
  - SimpleImputer
  - KNNImputer
  - IterativeImputer
  - MissingIndicator
  - missing values
  - missing value
  - NaN handling
  - missingness
  - MCAR
  - MAR
  - MNAR
  - missing completely at random
  - missing at random
  - missing not at random
  - row deletion
  - column deletion
  - listwise deletion
  - complete case analysis
up: "[[Poor-Quality Data]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

Most scikit-learn estimators refuse to fit on an array containing `NaN`, so every hole in $\mathbf{X}$ has to be resolved before training. Imputation fills the hole with a value derived from the rest of the data, keeping both the instance and the [[Feature]]. Reach for it over deletion when instances are scarce, when the column carries signal, or when production data may arrive with nulls the training data never had.

## Algorithm

A missing entry $x^{(i)}_j$ admits exactly three responses, each paying a different price:

1. **Drop the instance.** Row deletion, also called listwise deletion or complete case analysis. $m$ falls, and if the missingness is not random the survivors are a biased sample ([[Nonrepresentative Training Data]]). Defensible when only a few rows are affected and the holes look like accidents.
2. **Drop the feature.** Column deletion. Column $j$ leaves $\mathbf{X}$ for every instance, including the majority that had a good value. Defensible when the column is mostly holes, so there was little in it to keep.
3. **Impute.** Nothing is discarded, at the cost of inventing values and compressing the spread of column $j$.

Both deletion routes cost something, and which cost you pay is decided by why the entry is absent. Deleting when the hole depends on the value that is gone destroys information, because being missing was itself predictive and dropping the row or the column throws that signal away along with the blank. Deleting when the hole depends on some other column introduces bias, because the survivors are no longer representative on that column. So the mechanism is the first question, ahead of any choice of fill.

### Why a value is missing

Rubin (*Biometrika* 63(3), 1976) introduced missing at random and observed at random; the three-way classification below, and the labels MCAR and MNAR, are Little and Rubin's later systematization of it. Let $\mathbf{R}$ be the missingness indicator over the same $m \times n$ grid as $\mathbf{X}$, with $R_{ij} = 1$ when $x^{(i)}_j$ is observed and $0$ when it is not, and split the table into the part $\mathbf{R}$ reveals, $\mathbf{X}_{\text{obs}}$, and the part it hides, $\mathbf{X}_{\text{mis}}$. The mechanism is the distribution of $\mathbf{R}$ given both, carrying its own parameter $\psi$, and the three conditions are three ways that conditioning can collapse.

- **Missing completely at random (MCAR).** $P(\mathbf{R} \mid \mathbf{X}_{\text{obs}}, \mathbf{X}_{\text{mis}}, \psi) = P(\mathbf{R} \mid \psi)$. The holes fall independently of every value in the table, observed or not: someone simply skipped a field. The complete cases are then a uniformly drawn subsample of the full table, so row deletion costs sample size and nothing else.
- **Missing at random (MAR).** $P(\mathbf{R} \mid \mathbf{X}_{\text{obs}}, \mathbf{X}_{\text{mis}}, \psi) = P(\mathbf{R} \mid \mathbf{X}_{\text{obs}}, \psi)$. The hole depends on something in the table but not on the value that went missing: age goes unreported far more often for one survey group, and the group is a column you hold. Deletion now biases the survivors, since they are kept with a probability that varies by group, and conditioning on the observed columns can repair it, which is what a fill computed within group does.
- **Missing not at random (MNAR).** Neither identity holds, so after conditioning on everything observed $\mathbf{R}$ still depends on $\mathbf{X}_{\text{mis}}$. The value is missing because of itself, which is the income case: respondents who decline to state an income tend to have higher incomes than those who state one, so the undisclosed values are drawn from a shifted distribution and no column in the table says so. No procedure using observed data alone recovers that distribution, which is why MNAR is the case a better imputer does not fix. The fact of being missing is the only part of it that can be kept, and that is what an indicator column keeps.

**The mechanism is not testable from the data you hold.** MCAR can at least be argued against, since it predicts that complete and incomplete cases agree on every observed column. MAR against MNAR cannot be settled that way at all: the evidence that would separate them is exactly the values that are absent. Molenberghs, Beunckens, Sotto and Kenward (*JRSS-B* 70(2), 2008) make it sharp, showing that any MNAR model has an MAR counterpart reproducing the same observed-data likelihood, so the pair fits the file identically and differs only in what it says about entries nobody has. The mechanism is therefore an assumption argued from how the data was collected, who was asked what and who declined, and never a diagnosis read off a statistic.

MAR on its own is also not a licence to ignore the mechanism entirely. Rubin attaches a second condition, that $\psi$ be distinct from the parameter $\theta$ being estimated, and even then the licence is for direct-likelihood and Bayesian inference; sampling-distribution inference needs the stronger pair of MAR together with the observed data being observed at random, and stays conditional on the pattern of holes. How often MCAR actually holds is a judgement rather than a measured rate, and [[DMLS Ch05 Feature Engineering|DMLS chapter 5]] takes the sceptical side: blanks usually have a reason behind them, and finding the reason is part of choosing the response.

### Univariate fills

`SimpleImputer` takes the third route. At `fit` time it reduces each column to one statistic, at `transform` time it writes that statistic into every hole:

$$\tilde{x}_j = \operatorname{stat}\big\{\, x^{(i)}_j \;:\; i \in D_{\text{train}},\; x^{(i)}_j \text{ observed} \,\big\}$$

The learned vector is `imputer.statistics_`, in the [[Scikit-Learn Estimator API|fit/transform]] shape every transformer shares. That split carries the machine learning content. **The statistic is computed on the [[Training Set]] alone and reused unchanged afterwards.** A median taken over the full dataset, or recomputed on the [[Testing Set]], makes the fitted transformer a function of rows the deployed pipeline will never hold, and flatters every score that follows: that is [[Data Leakage]], which happens with nobody looking at anything, rather than [[Data Snooping Bias]]. Géron fits the imputer on *all* numeric columns, not only the one with holes, because future data is not guaranteed to be missing in the same places.

Four fills are on offer and they differ in what they do to the column. A **default constant** writes one literal everywhere, `mean` and `median` write the column's centre, and `most_frequent` writes its mode. `median` resists the extreme values that drag `mean` around in [[Skewed Data]]; `most_frequent` and `constant` are the only two that accept strings, which is why the categorical branch of a [[Pipeline]] imputes with `most_frequent` before [[One-Hot Encoding]].

One rule about the choice is worth stating precisely, because the usual version of it is wrong. The usual advice is to avoid filling with a value the variable could genuinely take, ex writing $0$ into a blank count of children, on the grounds that a real $0$ and a blank become the same row. The objection is that `mean`, `median` and `most_frequent` all write values the variable could take as well, so plausibility cannot be what is doing the work, and that objection is correct. What matters is not whether the fill is a value the column could hold but whether the fact of having been missing survives the fill, and no choice of statistic preserves it: a median-filled hole is as indistinguishable from a genuine median as a zero-filled hole is from a genuine zero. **What preserves it is a separate indicator column**, `add_indicator=True` or a standalone `MissingIndicator`, which records $1 - R_{ij}$ as a feature of its own and lets the model use the hole as evidence. So there are two independent decisions rather than one: keep a flag or lose the missingness, and then pick the statistic on what the fill does to the column's distribution, which is a question about variance and [[Correlation]] rather than about plausibility.

### Multivariate alternatives

- `KNNImputer` fills a hole from the $k$ most similar rows rather than from the column as a whole, using a nan-aware Euclidean distance that compares only coordinates neither row is missing, then averaging those neighbours' values.
- `IterativeImputer` models each incomplete column as a regression on all the others, round-robin: seed the holes with a baseline, fit an estimator for one column, rewrite its missing entries, repeat until the change falls below `tol` or `max_iter` rounds elapse. That is the MICE idea, more faithful to the joint distribution and far more expensive.
- `IterativeImputer` is experimental in scikit-learn 1.6 and has stayed experimental through 1.9, so `from sklearn.experimental import enable_iterative_imputer` has to run before the import succeeds.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `strategy` | | `"mean"` | moving `mean` $\to$ `median` makes the fill robust to outliers; `most_frequent` and `constant` extend to strings | `median` for numeric and skewed, `most_frequent` for categorical |
| `fill_value` | | `None` | only read when `strategy="constant"`; sets the literal written into every hole, and `None` means `0` for numeric columns and `"missing_value"` for string or object ones | a sentinel outside the data range, ex `-1` or `"missing"` |
| `add_indicator` | | `False` | turning it on appends one binary column per feature that had a hole at fit time, widening $\mathbf{X}$ | on whenever the missingness may be MNAR, since the flag is the only part of an MNAR hole that any fill can keep |
| `n_neighbors` (`KNNImputer`) | $k$ | `5` | smoother fills, less variance, more bias toward the global pattern, cost grows with $k$ | 3 to 10, chosen by [[Cross-Validation]] on the downstream score |
| `weights` (`KNNImputer`) | | `"uniform"` | `"distance"` lets closer rows dominate the average | `"distance"` when neighbour quality varies sharply |
| `max_iter` (`IterativeImputer`) | | `10` | more round-robin passes, closer to convergence, linear cost | raise only if the imputer warns that `tol` was not reached |

## Failure modes

- Fitting the imputer on the full dataset, or calling `fit_transform` on the test set, leaks held-out information into the learned median ([[Data Leakage]]).
- Imputing a categorical column with a mean. On raw strings `strategy="mean"` raises, but on a column already [[Ordinal Encoding|integer-coded]] it succeeds and writes a fractional code, a category that does not exist.
- Constant-fill compresses the column: replacing a fraction $p$ of it by its own mean leaves roughly $(1 - p)\sigma^2$ of the variance, weakening the [[Correlation]] with the target and making the feature look less useful than it is.
- Missing not at random: if `total_bedrooms` is blank precisely for the districts nobody surveyed, the blank is signal and median-filling erases it. `add_indicator=True` keeps it, and the standalone `MissingIndicator` produces the same flags on its own when the fill is handled elsewhere.
- Trusting `add_indicator` to flag a column that was complete during `fit`. The indicator is built from the features that had holes in the training split, so a column whose first `NaN` arrives at transform time gets no flag at all, and the hole is filled silently.
- Imputing before the train/test split, the same leak wearing a different hat.
- A hand-rolled fill written as `housing["total_bedrooms"].fillna(median, inplace=True)` is chained assignment: selecting the column hands back a temporary Series, so under copy-on-write (opt-in in pandas 2.x, the default from pandas 3.0) the write lands on that temporary, the frame is left untouched, and nothing warns. Reassign the column instead.

## Implementation

scikit-learn 1.6:

```python
import numpy as np
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")      # "mean" | "most_frequent" | "constant"
housing_num = housing.select_dtypes(include=[np.number])

imputer.fit(housing_num)        # learns one median per numeric column
imputer.statistics_             # the learned medians, reused at transform time
housing_imputed = imputer.transform(housing_num)   # NumPy array, same column order
```

Keeping a record of where the holes were:

```python
imputer = SimpleImputer(strategy="median", add_indicator=True)
# output gains one 0/1 column per feature that had a missing value at fit time

from sklearn.impute import MissingIndicator
flags = MissingIndicator().fit_transform(housing_num)   # the same flags, without any fill
```

Multivariate variants:

```python
from sklearn.impute import KNNImputer
knn_imp = KNNImputer(n_neighbors=5, weights="uniform")

from sklearn.experimental import enable_iterative_imputer  # noqa: F401  (required)
from sklearn.impute import IterativeImputer
iter_imp = IterativeImputer(max_iter=10, random_state=42)
```

The two deletion routes, for comparison (pandas 2.2 or later):

```python
housing = housing.dropna(subset=["total_bedrooms"])   # option 1: drop the rows
housing = housing.drop("total_bedrooms", axis=1)      # option 2: drop the column
median = housing["total_bedrooms"].median()           # option 3 by hand
housing["total_bedrooms"] = housing["total_bedrooms"].fillna(median)
```
