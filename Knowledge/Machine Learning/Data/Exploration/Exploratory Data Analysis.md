---
note_kind: method
aliases:
  - EDA
  - data exploration
  - exploratory analysis
  - exploring the data
  - data visualization
up: "[[Training Set]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Exploratory data analysis is the deliberate look you take at the data before fitting anything: row count, column types, missing values, categorical levels, the shape of every numeric distribution. Its output is a list of decisions, which columns need [[Missing Value Imputation]], which need [[One-Hot Encoding]], whether [[Feature Scaling]] is required at all. Do it at the start of every project and again whenever fresh data arrives; skip it and every preprocessing choice afterwards is a guess.

> [!warning]
> Explore a copy of the [[Training Set]] only. Géron's order is deliberate: glance at the structure just far enough to split sensibly, remove the [[Testing Set]], then explore properly. Whatever you learn from the test set leaks into your modelling choices and the estimate it later produces is optimistic. That is [[Data Snooping Bias]].

## Algorithm

1. `head()`: the first rows, so you see the columns and a real value.
2. `info()`: row count plus dtype and non-null count per column. A non-null count below the row count locates the missing values; an `object` dtype locates the non-numeric columns.
3. `value_counts()` on each non-numeric column: the levels and their frequencies. A level with a handful of instances will break a stratified split.
4. `describe()`: count, mean, standard deviation, min, the quartiles, max, for numeric columns.
5. `hist(bins=50)` on the whole frame: every numeric attribute at once, showing scale, range, and shape together in a way `describe()` cannot. Set `bins` well above the default of 10 at tens of thousands of rows, since coarse bins hide capping and multimodality, and move it until the shape stops changing.

Step 5 is where the [[California Housing|housing data]] confesses. `housing_median_age` and `median_house_value` are capped at their maximum, a spike in the last bin: a collection artifact, not a fact about houses, and a [[Model]] trained on it learns a price ceiling that does not exist. The attributes sit on wildly different scales, the argument for [[Feature Scaling]]. Several are long-tailed to the right, which is what [[Skewed Data]] and [[Feature Distribution Transformation]] address. And `median_income` is neither dollars nor raw: it is scaled and clipped, which no plot reveals and only the data documentation states.

## Hyperparameters

None for the plain fit.

## Failure modes

- Exploring the full dataset instead of the training set. The insight feels free; it has already contaminated the test estimate.
- Reading a histogram without checking the extreme bins for capped values or sentinel codes (`-1`, `9999`) standing in for "unknown".
- Trusting `describe()` when nulls are present. It drops them silently, so the count differs per column and each mean describes only the complete rows.
- Overplotting in a scatter plot. At full opacity a dense region and a merely populated one look identical; the structure appears only once `alpha` drops to 0.1 or 0.2 at tens of thousands of points, higher where the points are sparse.
- Stopping at the univariate view. Capping and skew show up per column; relationships between columns need [[Correlation]] and a scatter matrix.

## Implementation

pandas 2.x, matplotlib 3.x:

```python
import pandas as pd
import matplotlib.pyplot as plt

housing = strat_train_set.copy()   # explore a copy, never the original

housing.head()
housing.info()                     # dtypes and non-null counts
housing["ocean_proximity"].value_counts()
housing.describe()                 # numeric columns only, nulls excluded

housing.hist(bins=50, figsize=(12, 8))        # all numeric attributes
housing["median_income"].hist(bins=50)        # one attribute
plt.show()

# bivariate: a grid of pairwise scatter plots, then one pair up close
from pandas.plotting import scatter_matrix

attributes = ["median_house_value", "median_income", "total_rooms",
              "housing_median_age"]
scatter_matrix(housing[attributes], figsize=(12, 8))

housing.plot(kind="scatter", x="median_income", y="median_house_value",
             alpha=0.1, grid=True)
plt.show()
```

Géron's notebooks also call `save_fig(...)` after each plot. That is a helper defined in his companion repository, not part of matplotlib or pandas, and it raises `NameError` if pasted without being defined.
