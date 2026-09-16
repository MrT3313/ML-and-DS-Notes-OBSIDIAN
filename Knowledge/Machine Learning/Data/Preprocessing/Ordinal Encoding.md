---
note_kind: method
aliases:
  - ordinal encoder
  - OrdinalEncoder
  - label encoding
  - LabelEncoder
  - integer encoding
  - ordinal encode
up: "[[Feature Engineering]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Ordinal encoding replaces each category in a text column with one integer, turning a categorical [[Feature]] into a single numeric column. It is the cheapest categorical encoding: $K$ levels cost one column, where [[One-Hot Encoding]] costs $K$. Reach for it when the categories genuinely have an order (small, medium, large; cold, warm, hot) or when the model only compares values against thresholds, which is true of tree-based learners such as a decision tree or a random forest. Prefer one-hot everywhere else.

## Algorithm or formula

Fitting learns the sorted levels of each column, $\{c_0, \dots, c_{K-1}\}$, and transforming applies

$$g(c_k) = k, \qquad k = 0, 1, \dots, K-1$$

By default the levels are sorted lexicographically, an arbitrary order for nominal data, and are stored in `encoder.categories_`.

The map is the problem. One integer column carries both an order and a spacing, and $g$ invents both. Encoding ocean proximity as $0$ through $4$ tells a [[Linear Regression]] model that `NEAR BAY` (3) and `NEAR OCEAN` (4) are one step apart while `<1H OCEAN` (0) and `NEAR OCEAN` (4) are four steps apart, because that column's contribution is $\theta_j \, g(c)$, monotone and evenly spaced in $k$. Any distance-based method, k-nearest neighbors included, inherits the same fiction. A tree only asks whether $g(c) \le t$, so an arbitrary order costs it splits rather than correctness.

When an order does exist, pass it through `categories` rather than accepting the alphabetical default. For levels that may appear only after fitting, `handle_unknown="use_encoded_value"` with an explicit `unknown_value` maps them to a sentinel instead of raising. Note also that `OrdinalEncoder` is for features and expects 2D input, while `LabelEncoder` is for targets and expects 1D; they are routinely confused, and `LabelEncoder` does not belong inside a [[Column Transformer]].

`min_frequency` and `max_categories` reached `OrdinalEncoder` in scikit-learn 1.3, later than the `OneHotEncoder` versions of the same settings, and fold the rare levels of a feature into a single infrequent code.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `categories` | | `"auto"` | supplying a list fixes the level order, so $g$ encodes the real ranking instead of the alphabet | always set it by hand for genuinely ordered categories |
| `handle_unknown` | | `"error"` | `"use_encoded_value"` trades a hard failure at transform time for a sentinel code | set it whenever unseen levels are possible in production |
| `unknown_value` | | `None` | required with `"use_encoded_value"`; must sit outside $\{0,\dots,K-1\}$ | `-1`, or `np.nan` with a float `dtype` |
| `encoded_missing_value` | | `np.nan` | sets the code written for `NaN` inputs, so missing becomes its own level | an integer such as $K$ when you want missingness modelled |
| `min_frequency` / `max_categories` | | `None` | folding rare levels into one infrequent bucket shrinks $K$ | raise `min_frequency` when a long tail of levels appears a handful of times each |

## Failure modes

- Nominal categories fed to a linear or distance-based model: the invented order becomes a real coefficient, and the fit reports a trend across categories that means nothing.
- Alphabetical defaults on data that is actually ordered: `"high"`, `"low"`, `"medium"` sorts to $0, 1, 2$, which is not the ranking, and the mistake is invisible because nothing errors.
- An unseen level at transform time raises `ValueError` under the default `handle_unknown="error"`, so a model that trained fine crashes on the first production batch containing a new category.
- `LabelEncoder` used on a feature column: it takes only 1D input, so it must be applied column by column outside a [[Pipeline]], and the fitted mapping is easy to lose before test time.

## Implementation

scikit-learn 1.6:

```python
from sklearn.preprocessing import OrdinalEncoder

housing_cat = housing[["ocean_proximity"]]        # 2D, one column per feature

ordinal_encoder = OrdinalEncoder()
housing_cat_encoded = ordinal_encoder.fit_transform(housing_cat)
ordinal_encoder.categories_                        # the learned level order
```

With a real ranking and a safe fallback for unseen levels:

```python
import numpy as np

enc = OrdinalEncoder(
    categories=[["low", "medium", "high"]],        # the order that actually exists
    handle_unknown="use_encoded_value",
    unknown_value=np.nan,
    dtype=np.float64,
)
```
