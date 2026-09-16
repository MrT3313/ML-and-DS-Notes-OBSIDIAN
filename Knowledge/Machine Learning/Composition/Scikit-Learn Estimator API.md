---
note_kind: concept
aliases:
  - estimator
  - estimators
  - transformer
  - transformers
  - fit
  - fit_transform
  - scikit-learn API
  - sklearn API
  - Scikit-Learn Design
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

Scikit-Learn gives every object in the library one interface, so imputers, scalers, encoders and models are interchangeable parts. Anything that learns from data is an *estimator* and exposes `fit`; an estimator that also emits a reshaped dataset is a *transformer*; one that also emits predictions is a *predictor*.

## Formal statement

Not further quantitative at this depth. The API is an interface contract rather than a mathematical object, so what can be stated precisely is which methods each role owes its caller.

| role        | required                 | usually also                                                                 | example                           |
|-------------|--------------------------|------------------------------------------------------------------------------|-----------------------------------|
| estimator   | `fit(X, y=None)`         | `get_params()`, `set_params()`                                               | every object in the library       |
| transformer | `fit` and `transform(X)` | `fit_transform(X)`, `get_feature_names_out()`, sometimes `inverse_transform` | `SimpleImputer`, `StandardScaler` |
| predictor   | `fit` and `predict(X)`   | `score(X, y)`                                                                | `LinearRegression`                |

The roles nest: every transformer and every predictor is an estimator, and `fit_transform` is only an optimized shortcut for `fit` followed by `transform`. Unsupervised estimators ignore `y`, which is why a scaler and a regressor can sit in the same pipeline.

Four conventions hold the design together.

- **Hyperparameters go on the constructor**, stored unchanged under the same name as a public attribute. The constructor validates nothing and touches no data, so an unfitted estimator is a cheap, inspectable, copyable description of a recipe.
- **Learned parameters end in a trailing underscore** (`statistics_`, `mean_`, `coef_`). The convention is mechanical, not cosmetic: `check_is_fitted` tests for exactly such an attribute, and its absence is the `NotFittedError` raised when `transform` is called before `fit`.
- **`get_params` and `set_params` make the object introspectable**, which is how [[Grid Search]] reaches a [[Hyperparameter]] buried inside a nested pipeline through the double-underscore path `step__param`.
- **Datasets are NumPy arrays, SciPy sparse matrices or dataframes**, never a bespoke container class.

That last rule costs column names, since a dataframe becomes a bare array on the way through. Two later additions patch it: `get_feature_names_out()` reports the names a transformer produces, and `set_output(transform="pandas")` (scikit-learn 1.2 and later) makes `transform` hand back a dataframe.

## Where it is used

The contract is what makes composition possible. [[Pipeline]] chains estimators only because each answers to the same method names, and [[Column Transformer]] routes column subsets to separate branches for the same reason. Writing a [[Custom Transformer]] means implementing this contract by hand, which is where the constructor and underscore rules bite. Grid search relies on `get_params` to enumerate what is tunable and clones an unfitted estimator for each fold, which works only because constructors do no work. [[Missing Value Imputation]] and [[Feature Scaling]] supply the chapter's first transformers, [[Linear Regression]] its first predictor.
