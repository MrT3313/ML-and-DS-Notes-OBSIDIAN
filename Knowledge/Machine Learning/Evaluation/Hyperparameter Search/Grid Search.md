---
note_kind: method
aliases:
  - GridSearchCV
  - grid search
  - exhaustive search
  - HalvingGridSearchCV
up: "[[Model Selection]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Grid search takes a list of candidate values for each [[Hyperparameter]], forms their Cartesian product, scores every resulting combination by [[Cross-Validation]], and keeps the best. Reach for it when the space is small and discrete, when you already know roughly which region is worth combing, and when you want the candidate set fixed in advance so two runs examine exactly the same points. Once the space grows past a few hyperparameters, or once you suspect most of them barely move the score, [[Randomized Search]] spends the same budget better.

One requirement outranks the rest: the estimator handed to the search must be the whole [[Pipeline]], preprocessing included, so imputation and scaling are refit inside each fold. Searching over a model given already-transformed data lets the validation folds shape the transformer that is then scored on them, which is [[Data Leakage]] wearing a tuning loop as a disguise, the transformer being what gets contaminated rather than the choice of candidate.

## Algorithm

1. List the candidate values for every hyperparameter to be searched, as one dictionary, or as a list of dictionaries when some combinations make no sense jointly.
2. Form the Cartesian product of the values within each dictionary; every combination is a candidate.
3. Score every candidate by $K$-fold [[Cross-Validation]], the whole pipeline refit inside each fold.
4. Keep the candidate with the best mean fold score.
5. Refit the winner on the whole [[Training Set]] and evaluate it once on the [[Testing Set]].

For $k$ hyperparameters with $v$ values each and $K$ folds, the search costs

$$N_{\text{fits}} = v^{k} \cdot K + 1$$

the trailing $1$ being the refit of the winner on the whole [[Training Set]]. The exponent is the entire story: adding one three-valued hyperparameter triples the bill.

The worked housing grid is two dictionaries, $3 \times 3 = 9$ combinations and $2 \times 3 = 6$, so $15$ candidates at `cv=3`, giving $45$ fits plus the refit. A **list of dictionaries** is how you say "search these together, and separately search that other block", which keeps the search off combinations that make no sense jointly. The two blocks may overlap, and overlapping candidates are simply fitted twice.

The double-underscore names reach into nested objects: `preprocessing__geo__n_clusters` means the step called `preprocessing`, its inner transformer called `geo`, its `n_clusters` argument. This works because every estimator exposes `get_params` and `set_params` over that flattened namespace ([[Scikit-Learn Estimator API]]), which is what lets a search tune preprocessing and model in one space.

After fitting, `best_params_` holds the winning settings, `best_estimator_` the model already refit on all the training data (`refit=True`), `best_score_` the winner's mean cross-validation score, and `cv_results_` a dictionary that reads naturally as a dataframe, one row per candidate with per-fold scores.

> [!warning]
> `best_score_` is not an unbiased estimate of [[Generalization]] error. It is the maximum over many noisy candidate scores, and a maximum over noise is optimistically biased. Only the untouched [[Testing Set]] settles the real number.

Scikit-learn scorers always follow "higher is better", so error metrics are exposed negated: `scoring="neg_root_mean_squared_error"` reports minus the [[Root Mean Squared Error]], and a leading minus sign flips it back to a readable error.

### Successive halving

`HalvingGridSearchCV` is a tournament rather than a sweep. Every candidate is fitted on a small resource budget, the best $1 / \texttt{factor}$ survive, the budget is multiplied by `factor`, and the round repeats, so weak candidates are eliminated while they are still cheap. The resource is `n_samples` by default and can instead be any positive integer parameter such as a forest's `n_estimators`. The policy is due to Jamieson and Talwalkar, "Non-stochastic Best Arm Identification and Hyperparameter Optimization" (AISTATS 2016), and its budget-allocation refinement is Hyperband, Li, Jamieson, DeSalvo, Rostamizadeh and Talwalkar (*JMLR* 18, 2018), the two references scikit-learn itself cites for it. Seen from [[AutoML]] this is one performance estimation strategy and no more than that: it decides how a fixed budget is spread across the candidates it was handed, and it never proposes a candidate that was not written down.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `cv` | $K$ | 5 | less selection noise, linearly more fits | 3 while iterating on a large grid, 5 or 10 to decide |
| `scoring` | | estimator's own `score` | changes which candidate wins | name the metric the project is judged on |
| `refit` | | `True` | `best_estimator_` becomes usable directly | leave on unless refitting is prohibitive |
| `factor` (halving) | | 3 | fewer, harsher rounds; faster but likelier to kill a slow starter | 2 for a cautious search, 3 or 4 when compute is short |
| `min_resources` (halving) | | `'exhaust'` | first round more reliable, fewer rounds fit | raise it if early rounds look like coin flips |

`n_jobs` is not in the table: it changes wall-clock time and memory, never which candidate wins.

## Failure modes

- Combinatorial blowup. Five hyperparameters at four values each with `cv=5` is $5120$ fits: at a minute per fit, three and a half days.
- The optimum lands on a grid edge. The housing grid above returns `n_clusters=15`, the largest value offered, which means the grid was cut off before the score stopped improving. Re-centre on the boundary and run again: a randomized search over a wide range later reaches $45$ clusters and a clearly better score.
- The budget is spent on hyperparameters the model is insensitive to: every value of the one that matters is re-tested under settings of the one that does not, which is precisely the argument for sampling at random instead.
- Selection on a noisy score. With `cv=3` the per-fold spread can exceed the gap between the top candidates, so the winner may be the luckiest draw rather than the best one.
- Preprocessing fitted outside the search, which leaks and makes every score in `cv_results_` optimistic.

## Implementation

scikit-learn 1.6:

```python
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestRegressor

full_pipeline = Pipeline([
    ("preprocessing", preprocessing),
    ("random_forest", RandomForestRegressor(random_state=42)),
])

param_grid = [
    {"preprocessing__geo__n_clusters": [5, 8, 10],
     "random_forest__max_features": [4, 6, 8]},
    {"preprocessing__geo__n_clusters": [10, 15],
     "random_forest__max_features": [6, 8, 10]},
]

grid_search = GridSearchCV(full_pipeline, param_grid, cv=3,
                           scoring="neg_root_mean_squared_error")
grid_search.fit(housing, housing_labels)

grid_search.best_params_          # {'...n_clusters': 15, '...max_features': 6}
-grid_search.best_score_          # mean CV RMSE of the winner
cv_res = pd.DataFrame(grid_search.cv_results_)
```

Successive halving is still experimental, so the enabling import is mandatory and importing `HalvingGridSearchCV` without it raises `ImportError`:

```python
from sklearn.experimental import enable_halving_search_cv  # noqa: F401
from sklearn.model_selection import HalvingGridSearchCV

halving = HalvingGridSearchCV(full_pipeline, param_grid, cv=3, factor=3,
                              resource="n_samples", scoring="neg_root_mean_squared_error")
```
