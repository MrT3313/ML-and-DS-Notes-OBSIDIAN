---
note_kind: method
aliases:
  - pipelines
  - make_pipeline
  - sklearn Pipeline
  - sklearn.pipeline.Pipeline
  - preprocessing pipeline
up: "[[Scikit-Learn Estimator API]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

A pipeline bundles a sequence of transformers, plus an optional final predictor, into one estimator obeying the [[Scikit-Learn Estimator API]]. It exists for correctness, not convenience. Because the sequence is a single object with a single `fit`, nesting it inside [[Cross-Validation]] or [[Grid Search]] fits the preprocessing on the training folds alone. Running the same steps by hand on the full dataset before splitting lets a median or a category list computed from held-out rows leak into training, the ordinary route by which [[Data Leakage]] enters a project and makes every score optimistic. Reach for a pipeline the moment a preprocessing step *learns* anything.

The second reason is deployment: a fitted pipeline carries preprocessing and [[Model]] in one artifact, so serving cannot apply a different median than training did.

> [!warning]
> The word carries a second, unrelated sense in data engineering, the one [[Extract-Transform-Load]] carries: a *data pipeline* is a chain of data-processing stages run on a schedule, each pulling input from a data store, processing it, and writing its output back for a later stage to pick up. There the interface between stages is storage, so a stage can fail and be rerun without the stages around it being rerun too. A `sklearn.pipeline.Pipeline` is the in-memory version: ordinary objects in one process, passing arrays directly from call to call, the whole chain living and dying inside a single `fit`.

## Algorithm or formula

`fit(X, y)` calls `fit_transform` on every step **except the last**, feeding each output in as the next step's input, then calls `fit` on the last step with the fully transformed data. The final step is never asked for `transform`, which is why it may be a predictor that has no such method. The fitted pipeline then exposes whatever its final step exposes: `transform` and possibly `inverse_transform` for a transformer, `predict` and `score` for a predictor.

Two constructors. `Pipeline` takes explicit `(name, estimator)` tuples; `make_pipeline` takes bare estimators and auto-names each step from the lowercased class name (`simpleimputer`, `standardscaler`). The names are addresses, not decoration: they are what a search writes as `step_name__hyperparameter`. Steps are reachable as `pipe[-1]`, `pipe["standardize"]`, `pipe.named_steps.standardize`, and `pipe[:-1]` slices off a preprocessing-only sub-pipeline.

Every step but the last must be a transformer. Setting a step to `"passthrough"` (or `None`) disables it, which turns "should this step exist at all?" into something a search can answer. `memory="some/dir"` caches fitted transformers there, so a search that refits an expensive upstream step over identical inputs reuses the cached fit instead of recomputing it.

## Hyperparameters

None. The steps list is structure rather than a setting, and the hyperparameters belong to the steps themselves, reached through the `step__param` naming, which is where they are tuned.

## Failure modes

- A non-transformer in a non-final position: the pipeline raises at `fit` because it needs `transform` from every step except the last.
- Fitting a transformer outside the pipeline on the full dataset, then dropping the fitted object in. Cross-validation clones and refits it, so the hand-fit is wasted at best; if its output was used anywhere, the leak is back.
- Column names vanish once a step outputs a bare array, so importances read as `feature_12` and [[Feature Engineering]] becomes guesswork. `get_feature_names_out()` or `set_output(transform="pandas")` restores them.
- A step that silently receives the wrong columns still fits without complaint. Nothing checks that a scaler was handed numeric data; you find out from a bad score, not an exception.

## Implementation

scikit-learn 1.6:

```python
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression

num_pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("standardize", StandardScaler()),
])

# same thing, steps auto-named "simpleimputer" and "standardscaler"
num_pipeline = make_pipeline(SimpleImputer(strategy="median"), StandardScaler())

# preprocessing and model as one fittable, deployable object
lin_reg = make_pipeline(num_pipeline, LinearRegression())
lin_reg.fit(housing_num, housing_labels)
predictions = lin_reg.predict(housing_num)
```
