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
  - "[[HOML Ch03 Classification]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Scikit-Learn gives every object in the library one interface, so imputers, scalers, encoders and models are interchangeable parts. Anything that learns from data is an *estimator* and exposes `fit`; an estimator that also emits a reshaped dataset is a *transformer*; one that also emits predictions is a *predictor*.

## Formal statement

The API is an interface contract, and what it fixes precisely is which methods each role owes its caller.

| role        | required                 | usually also                                                                 | example                           |
|-------------|--------------------------|------------------------------------------------------------------------------|-----------------------------------|
| estimator   | `fit(X, y=None)`         | `get_params()`, `set_params()`                                               | every object in the library       |
| transformer | `fit` and `transform(X)` | `fit_transform(X)`, `get_feature_names_out()`, sometimes `inverse_transform` | `SimpleImputer`, `StandardScaler` |
| predictor   | `fit` and `predict(X)`   | `score(X, y)`                                                                | `LinearRegression`                |

The roles nest: every transformer and every predictor is an estimator, and `fit_transform` is only an optimized shortcut for `fit` followed by `transform`. Unsupervised estimators ignore `y`, which is why a scaler and a regressor can sit in the same pipeline.

### The response methods behind predict

The predictor row above understates what a classifier owes its caller, so take this subsection as that row corrected. `predict` is seldom the whole interface: underneath it sits a continuous response, and the label is only that response cut at a fixed threshold.

| method | what it returns | range |
|---|---|---|
| `decision_function(X)` | one uncalibrated signed score per instance. The sign gives the predicted class, the magnitude gives how far from the boundary the instance fell, so it reads as confidence and not as probability | all of $\mathbb{R}$ |
| `predict_proba(X)` | one estimated probability per class, calibrated only as well as the model happens to be. The contract fixes the range and the sum and requires nothing about those numbers matching observed frequencies; [[Model Calibration]] is where that "happens to be" is turned into a measurement | $[0, 1]$, each row summing to $1$ |
| `predict(X)` | the label, obtained by thresholding whichever of the two the estimator has | the class set |

Which of the two an estimator exposes is a property of that estimator, not of the contract. [[Stochastic Gradient Descent Classifier]] always has `decision_function`, and has `predict_proba` only for `loss="log_loss"` or `loss="modified_huber"`; under the default hinge loss there is no probability to report without wrapping the model in a separate calibration step, which at this interface is `CalibratedClassifierCV`, itself an estimator under this same contract and so composable like any other. `RandomForestClassifier` is the mirror image: `predict_proba` is there, computed from the class votes of its trees, and `decision_function` does not exist on it at all.

This is not API trivia. Anything that sweeps a threshold needs a continuous response to sweep, so a [[ROC Curve]] and a [[Precision-Recall Tradeoff]] are built from whichever of the two the estimator provides, and code that compares two classifiers has to ask each one for the method it actually has rather than assume they share one. That is why a random forest has to be scored through the positive-class column of `predict_proba` while a linear classifier is scored from `decision_function` directly, and why the generic `cross_val_predict` takes the response method as an argument instead of fixing it.

### Design conventions

Four conventions hold the design together.

- **Hyperparameters go on the constructor**, stored unchanged under the same name as a public attribute. The constructor validates nothing and touches no data, so an unfitted estimator is a cheap, inspectable, copyable description of a recipe.
- **Learned parameters end in a trailing underscore** (`statistics_`, `mean_`, `coef_`). The convention is mechanical, not cosmetic: `check_is_fitted` tests for exactly such an attribute, and its absence is the `NotFittedError` raised when `transform` is called before `fit`.
- **`get_params` and `set_params` make the object introspectable**, which is how [[Grid Search]] reaches a [[Hyperparameter]] buried inside a nested pipeline through the double-underscore path `step__param`.
- **Datasets are NumPy arrays, SciPy sparse matrices or dataframes**, never a bespoke container class.

That last rule costs column names, since a dataframe becomes a bare array on the way through. Two later additions patch it: `get_feature_names_out()` reports the names a transformer produces, and `set_output(transform="pandas")` (scikit-learn 1.2 and later) makes `transform` hand back a dataframe.

## Where it is used

The contract is what makes composition possible. [[Pipeline]] chains estimators only because each answers to the same method names, and [[Column Transformer]] routes column subsets to separate branches for the same reason. Writing a [[Custom Transformer]] means implementing this contract by hand, which is where the constructor and underscore rules bite. Grid search relies on `get_params` to enumerate what is tunable and clones an unfitted estimator for each fold, which works only because constructors do no work. [[Missing Value Imputation]] and [[Feature Scaling]] are the first transformers most projects meet, [[Linear Regression]] the first predictor.
