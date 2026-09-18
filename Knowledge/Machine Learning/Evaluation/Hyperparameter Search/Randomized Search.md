---
note_kind: method
aliases:
  - RandomizedSearchCV
  - random search
  - randomised search
  - HalvingRandomSearchCV
up: "[[Model Selection]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Randomized search draws a fixed number of [[Hyperparameter]] combinations at random, from explicit lists or from continuous distributions, scores each by [[Cross-Validation]], and keeps the best. Reach for it whenever the space is large, continuous, or only partly understood, which in practice is most of the time. Its decisive property is that the budget `n_iter` is set independently of how many hyperparameters are in play: widening the search costs nothing extra, and the run can be stopped whenever the compute runs out, whereas [[Grid Search]] must finish its product or finish nothing.

As with any search, the estimator passed in is the whole [[Pipeline]], so preprocessing is refit inside each fold rather than once on data that includes the validation rows ([[Data Snooping Bias]]).

## Algorithm

1. Give every hyperparameter to be searched a list or a distribution to draw from; together they define $P$ over the hyperparameter space.
2. Fix the budget $n_{\text{iter}}$ and a [[Random Seed]], so the candidate set can be reproduced.
3. For $t = 1, \dots, n_{\text{iter}}$, sample $\boldsymbol\lambda_t \sim P$.
4. Score each $\boldsymbol\lambda_t$ with $K$-fold [[Cross-Validation]], the whole pipeline refit inside each fold.
5. Return $\arg\min_t \hat{\mathcal{L}}_{\text{CV}}(\boldsymbol\lambda_t)$, refit on the whole [[Training Set]], and evaluate once on the [[Testing Set]].

The cost is $n_{\text{iter}} \cdot K + 1$ fits, flat in the dimension of the space.

The reason this beats a grid is an argument from *effective dimension*, due to Bergstra and Bengio, "Random Search for Hyper-Parameter Optimization" (JMLR, 2012). Suppose only one of $k$ hyperparameters actually moves the score. A grid of $v$ values per hyperparameter tries only $v$ distinct values of that one, re-testing each $v^{k-1}$ times under settings that do not matter. Random search tries a *different* value of every hyperparameter on every trial, so $n_{\text{iter}}$ draws give $n_{\text{iter}}$ distinct values of the one that counts. A related consequence is easy to state: if a fraction $p$ of the space is "good", the chance of missing it entirely in $n$ independent draws is $(1-p)^{n}$, so $60$ draws leave under a $5\%$ chance of missing the best $5\%$ of the space.

### Distributions, not just lists

A value may be given as a list, which is sampled uniformly, or as any object with an `rvs` method, which is what `scipy.stats` supplies. `randint(low, high)` samples integers and is **high-exclusive**, so `randint(low=3, high=50)` never returns $50$. `uniform(loc, scale)` covers a bounded continuous range. `loguniform(a, b)` is the one worth deliberate thought: for anything spanning orders of magnitude, such as a regularization strength, a [[Learning Rate]], or an SVM's $C$, uniform sampling on $[a, b]$ puts almost every draw in the top decade, while log-uniform sampling spreads them evenly across decades, which is how such parameters actually behave.

`HalvingRandomSearchCV` composes the two savings: sample candidates at random, then eliminate them in successive-halving rounds of growing resource. When even that is too slow, the next step up is Bayesian optimization, which fits a surrogate model to the scores seen so far and samples where the expected improvement is highest, rather than sampling blindly.

Scoring follows the "higher is better" convention described under grid search, so error metrics arrive negated and `-search.best_score_` is the readable [[Root Mean Squared Error]].

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `n_iter` | $n$ | 10 | better coverage, linearly more fits | as many as the time budget allows; 10 is a smoke test, 50 to 100 is a search |
| `cv` | $K$ | 5 | less selection noise, linearly more fits | 3 while exploring, 5 or 10 to decide |
| `scoring` | | estimator's own `score` | changes which candidate wins | name the metric the project is judged on |
| `refit` | | `True` | `best_estimator_` comes back fitted on all training data | leave on |
| `factor` (halving) | | 3 | fewer, harsher elimination rounds | 2 to 4 |

`n_jobs` is not in the table: it changes wall-clock time and memory, never which candidate wins. `random_state` is left out on purpose: it fixes which candidates are sampled from the distributions, so it changes which candidate comes back, but it is pinned rather than tuned. Set it to an integer, see [[Random Seed]].

## Failure modes

- A budget too small for the space. Ten draws over six hyperparameters is closer to a lucky guess than a search; the `n_iter=10` in the implementation below is a demonstration, not a recommendation.
- A badly chosen distribution. Uniform sampling over $[10^{-5}, 10^{-1}]$ puts roughly $99.99\%$ of draws in the last decade, so the small values are never really tried. Use `loguniform` for scale parameters.
- Bounds that exclude the optimum. The search reports the best of what it drew, and a support stopping short of the good region looks exactly like a search that converged.
- Irreproducibility. Without `random_state` the candidate set differs every run, so two runs disagree and neither can be re-examined ([[Random Seed]]).
- Reading `best_score_` as a generalization estimate. It is a minimum over many noisy cross-validation means and is optimistically biased for the same reason grid search's is, which is what the [[Testing Set]] is held back to correct.

## Implementation

scikit-learn 1.6 with SciPy 1.14:

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, loguniform

param_distribs = {
    "preprocessing__geo__n_clusters": randint(low=3, high=50),   # 3..49
    "random_forest__max_features": randint(low=2, high=20),
}

rnd_search = RandomizedSearchCV(
    full_pipeline, param_distributions=param_distribs, n_iter=10, cv=3,
    scoring="neg_root_mean_squared_error", random_state=42)

rnd_search.fit(housing, housing_labels)
final_model = rnd_search.best_estimator_   # preprocessing included
```

A scale parameter belongs on a log scale:

```python
param_distribs = {"svr__kernel": ["linear", "rbf"],
                  "svr__C": loguniform(20, 200_000)}
```

Successive halving remains experimental, so the enabling import is required:

```python
from sklearn.experimental import enable_halving_search_cv  # noqa: F401
from sklearn.model_selection import HalvingRandomSearchCV
```
