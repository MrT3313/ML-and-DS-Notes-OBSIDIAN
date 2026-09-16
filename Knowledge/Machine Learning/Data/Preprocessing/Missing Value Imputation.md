---
note_kind: method
aliases:
  - imputation
  - impute
  - imputer
  - SimpleImputer
  - KNNImputer
  - IterativeImputer
  - missing values
  - missing value
  - NaN handling
up: "[[Poor-Quality Data]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Most scikit-learn estimators refuse to fit on an array containing `NaN`, so every hole in $\mathbf{X}$ has to be resolved before training. Imputation fills the hole with a value derived from the rest of the data, keeping both the instance and the [[Feature]]. Reach for it over deletion when instances are scarce, when the column carries signal, or when production data may arrive with nulls the training data never had.

## Algorithm

A missing entry $x^{(i)}_j$ admits exactly three responses, each paying a different price:

1. **Drop the instance.** $m$ falls, and if the missingness is not random the survivors are a biased sample ([[Nonrepresentative Training Data]]).
2. **Drop the feature.** Column $j$ leaves $\mathbf{X}$ for every instance, including the majority that had a good value.
3. **Impute.** Nothing is discarded, at the cost of inventing values and compressing the spread of column $j$.

`SimpleImputer` takes the third route. At `fit` time it reduces each column to one statistic, at `transform` time it writes that statistic into every hole:

$$\tilde{x}_j = \operatorname{stat}\big\{\, x^{(i)}_j \;:\; i \in D_{\text{train}},\; x^{(i)}_j \text{ observed} \,\big\}$$

The learned vector is `imputer.statistics_`, in the [[Scikit-Learn Estimator API|fit/transform]] shape every transformer shares. That split carries the machine learning content. **The statistic is computed on the [[Training Set]] alone and reused unchanged afterwards.** A median taken over the full dataset, or recomputed on the [[Testing Set]], folds held-out rows into the fitted transformer and flatters every score that follows: [[Data Snooping Bias]]. Geron fits the imputer on *all* numeric columns, not only the one with holes, because future data is not guaranteed to be missing in the same places.

Which statistic: `median` resists the extreme values that drag `mean` around in [[Skewed Data]]; `most_frequent` and `constant` are the only two that accept strings, which is why the categorical branch of a [[Pipeline]] imputes with `most_frequent` before [[One-Hot Encoding]].

### Multivariate alternatives

- `KNNImputer` fills a hole from the $k$ most similar rows rather than from the column as a whole, using a nan-aware Euclidean distance that compares only coordinates neither row is missing, then averaging those neighbours' values.
- `IterativeImputer` models each incomplete column as a regression on all the others, round-robin: seed the holes with a baseline, fit an estimator for one column, rewrite its missing entries, repeat until the change falls below `tol` or `max_iter` rounds elapse. That is the MICE idea, more faithful to the joint distribution and far more expensive. 
- `IterativeImputer` is still experimental as of scikit-learn 1.9, so `from sklearn.experimental import enable_iterative_imputer` has to run before the import succeeds.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `strategy` | | `"mean"` | moving `mean` $\to$ `median` makes the fill robust to outliers; `most_frequent` and `constant` extend to strings | `median` for numeric and skewed, `most_frequent` for categorical |
| `fill_value` | | `None` | only read when `strategy="constant"`; sets the literal written into every hole | a sentinel outside the data range, ex `0` or `"missing"` |
| `add_indicator` | | `False` | turning it on appends one binary column per incomplete feature, widening $\mathbf{X}$ | on when missingness itself may predict the target |
| `n_neighbors` (`KNNImputer`) | $k$ | `5` | smoother fills, less variance, more bias toward the global pattern, cost grows with $k$ | 3 to 10, chosen by [[Cross-Validation]] on the downstream score |
| `weights` (`KNNImputer`) | | `"uniform"` | `"distance"` lets closer rows dominate the average | `"distance"` when neighbour quality varies sharply |
| `max_iter` (`IterativeImputer`) | | `10` | more round-robin passes, closer to convergence, linear cost | raise only if the imputer warns that `tol` was not reached |

## Failure modes

- Fitting the imputer on the full dataset, or calling `fit_transform` on the test set, leaks held-out information into the learned median.
- Imputing a categorical column with a mean. On raw strings `strategy="mean"` raises, but on a column already [[Ordinal Encoding|integer-coded]] it succeeds and writes a fractional code, a category that does not exist.
- Constant-fill compresses the column: replacing a fraction $p$ of it by its own mean leaves roughly $(1 - p)\sigma^2$ of the variance, weakening the [[Correlation]] with the target and making the feature look less useful than it is.
- Missing not at random: if `total_bedrooms` is blank precisely for the districts nobody surveyed, the blank is signal and median-filling erases it. `add_indicator=True` keeps it, and the standalone `MissingIndicator` produces the same flags on its own when the fill is handled elsewhere.
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
