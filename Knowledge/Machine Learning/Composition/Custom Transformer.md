---
note_kind: method
aliases:
  - custom transformers
  - FunctionTransformer
  - TransformerMixin
  - BaseEstimator
  - custom estimator
  - sklearn.utils.validation
up: "[[Scikit-Learn Estimator API]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

The built-in transformers cover imputation, scaling and encoding. Anything else you invent during [[Feature Engineering]], a ratio between two columns, a log of a skewed one, a distance to a landmark, must become a transformer too, or it cannot sit inside a [[Pipeline]] and so will not be fit on training folds only. Writing one is how a hand-rolled idea earns the same leakage safety and deployability as a built-in. Take the lightest route that works: a stateless function needs no class, a transformation that *learns* at fit time does.

## Algorithm or formula

**Route 1, stateless: `FunctionTransformer`.** Wrap a callable. Nothing is learned, so `fit` only records `n_features_in_` and `transform` calls the function. Supply `inverse_func` when the transformation must be undone, which is what makes a log feature or a log target reversible ([[Feature Distribution Transformation]], [[Target Scaling]]), and `feature_names_out=` (a callable, or `"one-to-one"`) so the outputs keep names.

**Route 2, stateful: subclass `BaseEstimator, TransformerMixin`.** `TransformerMixin` supplies `fit_transform` and `set_output` support; `BaseEstimator` supplies `get_params` and `set_params`, which is what makes the transformer tunable by [[Grid Search]]. Four rules, all routinely broken:

- The constructor takes explicit keyword arguments only, no `*args` or `**kwargs`, and assigns each unchanged to an attribute of the same name. Nothing else happens in it, not even validation, because `get_params` reads the object back through those attributes.
- `fit` returns `self`; everything it learns ends in a trailing underscore.
- `transform` calls `check_is_fitted(self)` first, which is what raises the `NotFittedError` users see.

**Route 3: wrap a whole estimator.** A transformer may fit another model inside itself. The chapter's worked example, `ClusterSimilarity`, fits a $k$-means clustering ([[Clustering]]) on latitude and longitude at fit time, keeps the centres, and at transform time emits an [[Feature Distribution Transformation|RBF similarity]] from each instance to each centre, turning two raw coordinates into $k$ smooth features that a linear model can actually use. What makes it route 3 rather than route 1 is only where the landmarks come from: they are fitted, not hand-picked, so the object has state to carry.

### Validation

`sklearn.utils.validation` holds the checks a well-behaved estimator owes its caller. `check_array` coerces and checks one input array. `validate_data(self, X, reset=True)` (public since scikit-learn 1.6, previously the private `BaseEstimator._validate_data`) does that *and* records `n_features_in_` and `feature_names_in_`; calling it again in `transform` with `reset=False` makes mismatched input fail loudly rather than produce nonsense. `check_is_fitted(self)` looks for a trailing-underscore attribute and raises `NotFittedError` if there is none. Define `get_feature_names_out()` too, so invented names survive a [[Column Transformer]] and importances stay readable.

## Hyperparameters

Whatever you put on the constructor, and nothing else: that is the point of the rule. The chapter's example carries these.

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `n_clusters` | $k$ | `10` | more centres, more output features, finer spatial resolution, higher variance | search it with the model attached, since the useful $k$ depends on the downstream regressor |
| `gamma` | $\gamma$ | `1.0` | passed straight through to `rbf_kernel`, with the usual effect of the RBF width; because there are $k$ landmarks rather than one, too large a value leaves an instance near-zero on every output column at once | search jointly with `n_clusters` on a log scale |
| `random_state` | | `None` | fixes the clustering initialization | pin it so runs are reproducible |

## Failure modes

- The constructor does work: validating an argument, converting a list to an array, renaming a parameter. `get_params` then no longer reconstructs the object, so `clone` inside cross-validation silently builds a different transformer than the one you tested.
- `fit` forgets `return self`, so `fit_transform` and every pipeline step downstream receive `None`.
- A learned attribute stored without the trailing underscore: `check_is_fitted` cannot see it, `NotFittedError` never fires, and an unfitted transformer runs on production data.
- No width or name check at transform time, so a frame whose columns arrive in a different order transforms without complaint and every feature is wrong.
- Fitting on the target, or on whole-dataset statistics computed in the constructor, reintroduces the leakage the pipeline was built to prevent.

## Implementation

scikit-learn 1.6:

```python
import numpy as np
from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.cluster import KMeans
from sklearn.metrics.pairwise import rbf_kernel
from sklearn.preprocessing import FunctionTransformer
from sklearn.utils.validation import check_is_fitted, validate_data

# route 1: stateless, invertible, named
log_transformer = FunctionTransformer(
    np.log, inverse_func=np.exp, feature_names_out="one-to-one")

# route 2 and 3: learns at fit time by fitting another estimator
class ClusterSimilarity(TransformerMixin, BaseEstimator):
    def __init__(self, n_clusters=10, gamma=1.0, random_state=None):
        self.n_clusters = n_clusters
        self.gamma = gamma
        self.random_state = random_state

    def fit(self, X, y=None, sample_weight=None):
        X = validate_data(self, X)                      # sets n_features_in_
        self.kmeans_ = KMeans(self.n_clusters, n_init=10,
                              random_state=self.random_state)
        self.kmeans_.fit(X, sample_weight=sample_weight)
        return self

    def transform(self, X):
        check_is_fitted(self)                           # raises NotFittedError
        X = validate_data(self, X, reset=False)         # rejects wrong width
        return rbf_kernel(X, self.kmeans_.cluster_centers_, gamma=self.gamma)

    def get_feature_names_out(self, names=None):
        return np.array([f"cluster_{i}_similarity" for i in range(self.n_clusters)])
```
