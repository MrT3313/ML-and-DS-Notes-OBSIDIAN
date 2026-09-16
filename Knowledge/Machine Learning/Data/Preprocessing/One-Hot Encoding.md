---
note_kind: method
aliases:
  - one hot encoding
  - one-hot
  - OneHotEncoder
  - dummy variables
  - dummy encoding
  - indicator variables
up: "[[Feature Engineering]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

One-hot encoding replaces a categorical [[Feature]] with $K$ binary columns, one per level, exactly one of them hot in any row. It is the default for nominal categories because, unlike [[Ordinal Encoding]], it invents no order: no level is nearer to any other, so a [[Linear Regression]] model learns an independent coefficient per level. The cost is width, and it stops being the obvious choice once $K$ runs into the hundreds.

## Algorithm or formula

Fitting records the levels $\{c_0, \dots, c_{K-1}\}$ of each column; transforming maps a level to the matching standard basis vector,

$$g(c_k) = \mathbf{e}_k \in \{0,1\}^{K}, \qquad \sum_{j=0}^{K-1} g(c)_j = 1$$

so a column of $m$ strings becomes an $m \times K$ block of $\mathbf{X}$. Because almost every entry is 0, the output is a SciPy sparse matrix by default, which is why printing `fit_transform`'s result shows a sparse representation rather than anything dataframe-shaped. Call `.toarray()`, or pass `sparse_output=False` (the parameter was named `sparse` before scikit-learn 1.2), when a dense array is actually wanted.

The constant sum above is also the catch. The $K$ indicators add up to the all-ones column, so with an intercept the design matrix is rank deficient: the dummy variable trap, which leaves a linear model's individual coefficients unidentifiable. `drop="first"` removes one level per feature and restores full rank, `drop="if_binary"` does so only where $K = 2$. Trees are indifferent to the collinearity and keep the full set.

Two settings matter downstream. `handle_unknown="ignore"` encodes a level unseen at fit time as all zeros instead of raising, which keeps a fitted [[Pipeline]] alive on fresh production data. `get_feature_names_out()` returns names such as `ocean_proximity_INLAND`, which is how the expanded columns are matched back to feature importances later.

`min_frequency` and `max_categories`, both available since scikit-learn 1.1, fold the rare levels of a feature into a single infrequent column instead of giving each one its own.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `handle_unknown` | | `"error"` | `"ignore"` turns an unseen level into an all-zero row block instead of an exception | `"ignore"` for any pipeline that will be refitted or deployed |
| `sparse_output` | | `True` | setting `False` densifies the block, costing $m \times K$ floats of memory | leave `True` unless a downstream step rejects sparse input |
| `drop` | | `None` | `"first"` removes one column per feature, cutting width by the number of features and breaking the collinearity | `"first"` for linear models with an intercept, `None` for trees |
| `min_frequency` | | `None` | raising it folds more rare levels into a single infrequent column, shrinking $K$ | set to a count or a fraction when a long tail of levels is near-empty |
| `max_categories` | | `None` | caps the number of output columns per feature, bucketing the remainder as infrequent | a few dozen at most, chosen against the downstream score |
| `categories` | | `"auto"` | fixes the level set and column order rather than inferring it from the training data | set explicitly when the full level set is known in advance |

## Failure modes

- High cardinality: a ZIP code column with thousands of levels becomes thousands of near-empty columns, inflating memory, slowing the fit, and leaving each level too few instances to estimate its coefficient. This is the standard motivation for target encoding or a learned embedding.
- The default `handle_unknown="error"` crashes at transform time on any level absent when the encoder was fitted, exactly the case a live model meets first.
- Keeping all $K$ columns alongside an intercept in a linear model: the fit runs, but the coefficients are not uniquely determined and reading one as the effect of that level is wrong.
- Forgetting the output is sparse and calling a dense-only operation on it, or densifying a wide block and exhausting memory on data that fit comfortably while sparse.
- Reaching for `pandas.get_dummies` instead: it gets the training frame right but remembers nothing, so a test frame missing one level silently yields a different number of columns.

## Implementation

scikit-learn 1.6:

```python
from sklearn.preprocessing import OneHotEncoder

housing_cat = housing[["ocean_proximity"]]         # 2D input

cat_encoder = OneHotEncoder()                      # sparse_output=True by default
housing_cat_1hot = cat_encoder.fit_transform(housing_cat)

housing_cat_1hot.toarray()                         # densify when needed
cat_encoder.categories_                            # levels learned at fit time
cat_encoder.get_feature_names_out()                # ['ocean_proximity_<1H OCEAN', ...]
```

Inside the categorical branch of a preprocessing [[Pipeline]], paired with [[Missing Value Imputation]]:

```python
from sklearn.pipeline import make_pipeline
from sklearn.impute import SimpleImputer

cat_pipeline = make_pipeline(
    SimpleImputer(strategy="most_frequent"),       # one of the two strategies that accept strings
    OneHotEncoder(handle_unknown="ignore"),
)
```
