---
note_kind: method
aliases:
  - ColumnTransformer
  - column transformers
  - make_column_transformer
  - make_column_selector
  - column selector
up: "[[Pipeline]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Numeric and categorical columns need incompatible treatment, a median fill and a scaler for one, a most-frequent fill and [[One-Hot Encoding]] for the other, yet the model needs them back as one matrix. A `ColumnTransformer` is the branching step: it routes named subsets of columns to their own [[Pipeline]], fits each branch independently, and concatenates the results horizontally. Reach for it whenever the dataset is heterogeneous, which for tabular data is almost always; a plain pipeline cannot do it, because it applies each step to everything it is given. The result is itself a transformer under the [[Scikit-Learn Estimator API]], so the usual move is to make it the first step of an outer pipeline whose last step is the model.

## Algorithm or formula

Each entry is a `(name, transformer, columns)` triple. `columns` is a list of names, a list of positions, a boolean mask, or a callable selector. In place of a transformer the strings `"drop"` and `"passthrough"` are accepted, meaning discard these columns or forward them untouched.

`fit` runs each branch on its own subset; `transform` runs them all and stacks the outputs left to right **in the order the transformers were listed**, not the order the columns appeared in the input. Reading `get_feature_names_out()` without knowing that is how importances get mismatched to features.

Columns named in no entry are governed by `remainder`, which defaults to `"drop"`. Set `remainder="passthrough"` to keep them; passthrough columns are appended at the right of the output.

`make_column_transformer` is the unnamed constructor, taking `(transformer, columns)` pairs and auto-naming each branch, the same trade as `make_pipeline`. `make_column_selector` replaces the explicit column list with a dtype predicate (`dtype_include=np.number`, `dtype_include=object`), so a column added later is picked up automatically instead of silently dropped.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `remainder` | | `"drop"` | `"passthrough"` keeps unlisted columns instead of discarding them; a transformer applies that transformer to them | set it deliberately, never by accident |

## Failure modes

- `remainder="drop"` is the default and it is silent. A column you forgot to list disappears from the matrix, the fit succeeds, and the only symptom is a slightly worse score.
- Overlapping column lists across branches: the column is transformed twice and appears twice in the output.
- Passing a single column as the string `"ocean_proximity"` rather than the one-element list `["ocean_proximity"]`. The string form hands the branch a 1D Series, the list form a 2D frame. Encoders that require 2D input raise on the first, and the error points at the transformer rather than the missing bracket.
- Output order follows the transformer list, so appending a branch shifts every downstream index and invalidates hard-coded positions.
- A dtype selector matching nothing produces an empty branch rather than an error, so verify what each selector actually caught.

## Implementation

scikit-learn 1.6:

```python
import numpy as np
from sklearn.compose import ColumnTransformer, make_column_transformer, make_column_selector
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

num_attribs = ["longitude", "latitude", "housing_median_age", "total_rooms",
               "total_bedrooms", "population", "households", "median_income"]
cat_attribs = ["ocean_proximity"]

num_pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("standardize", StandardScaler()),
])
cat_pipeline = make_pipeline(
    SimpleImputer(strategy="most_frequent"),
    OneHotEncoder(handle_unknown="ignore"))

preprocessing = ColumnTransformer([
    ("num", num_pipeline, num_attribs),
    ("cat", cat_pipeline, cat_attribs),
])

# same thing, selecting by dtype instead of naming every column
preprocessing = make_column_transformer(
    (num_pipeline, make_column_selector(dtype_include=np.number)),
    (cat_pipeline, make_column_selector(dtype_include=object)),
)

preprocessing.fit_transform(housing)
preprocessing.get_feature_names_out()
```
