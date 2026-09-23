---
note_kind: method
aliases:
  - learning curves
  - learning curve
  - learning_curve
  - LearningCurveDisplay
up: "[[Generalization]]"
sources:
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

A learning curve plots training error and validation error on the same axes, against how much the model has been given. Reach for it when a single score says the model is not good enough and you need to know *why*, because the two repairs point in opposite directions: more data, or a different model. One number cannot tell those apart and two curves can.

Two different quantities travel under the same name, and conflating them is the usual mistake.

**Indexed by training set size.** The estimator is refit from scratch on $m_1 < m_2 < \dots < m_T$ instances, and at each size both its training error and its held-out error are recorded. This is the capacity diagnostic, and it is what `sklearn.model_selection.learning_curve` computes; the parameter it sweeps is literally called `train_sizes`. It answers "would more data help".

**Indexed by training iteration.** One estimator is fit incrementally and its training and validation error are recorded after each [[Epoch]]. Nothing is refit and the training set never changes. It answers "have I trained long enough, and have I started training too long". That curve is a convergence diagnostic and belongs to [[Early Stopping]], which carries the loop that produces it. `learning_curve` cannot produce it, because it refits at every point and never exposes an intermediate state. The rest of this note is the size-indexed curve.

## Algorithm

1. Pick the sizes $m_1 < m_2 < \dots < m_T$ you want the curve evaluated at, up to the largest training set a fold can give you.
2. Pick a [[Cross-Validation]] splitter with $K$ folds. Every point on the curve is a cross-validated score, not a single fit, which is what keeps the curve readable rather than noise.
3. For each fold $k$ and each size $t$: take the first $m_t$ instances of fold $k$'s training indices as the subset $S_{k,t}$, fit $h_{k,t}$ on $S_{k,t}$ alone, and score it twice, once on $S_{k,t}$ and once on the held-out fold $F_k$.
4. Average each over the folds:

$$\bar{e}_{\text{train}}(m_t) = \frac{1}{K}\sum_{k=1}^{K} \mathcal{L}\big(h_{k,t},\, S_{k,t}\big), \qquad \bar{e}_{\text{val}}(m_t) = \frac{1}{K}\sum_{k=1}^{K} \mathcal{L}\big(h_{k,t},\, F_k\big)$$

5. Plot both against $m_t$ and read the shape.

The cost is $K \times T$ fits, and the subsets are nested prefixes, so $S_{k,1} \subset S_{k,2} \subset \dots \subset S_{k,T}$.

### Reading the shape

| shape at the right edge | reading | what to do |
|---|---|---|
| both curves flat, close together, and high | [[Underfitting]]. The model cannot represent the pattern, and it is already saturated on the data it has | change the model family, add [[Feature\|features]], or weaken the constraint. More data is wasted |
| training error low and flat, validation error well above it, gap not closing | [[Overfitting]]. The model is fitting instance-specific noise | more data, [[Regularization]], or fewer degrees of freedom |
| gap still visibly narrowing at the largest size | not yet converged | more data is the cheapest move available, and the curve says roughly how much would buy how much |

Only the right edge is worth reading. At $m_t = 1$ both quantities are degenerate, since almost any model fits one or two instances exactly, so the training curve starts near $0$ and climbs while the validation curve starts terrible and falls.

The first two shapes are exactly the two ends of the [[Bias-Variance Tradeoff]], which is what makes the picture a capacity diagnostic rather than a progress report. The worked pair in [[HOML Ch04 Training Models|HOML chapter 4]] runs the identical procedure on one quadratic dataset of $100$ points at `cv=5`: a plain [[Linear Regression]] gives the first shape, both curves settling onto a plateau at an [[Root Mean Squared Error|RMSE]] just under $1.75$ with barely any gap, and a degree-$10$ [[Polynomial Regression]] pipeline on the same data gives the second, training error noticeably lower and a gap that is still open at $80$ instances.

The flat-and-high verdict does not say the model is bad; it says that *this* model has stopped being data-limited, which is the honest answer to [[Insufficient Training Data]] and the only empirical one available.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `train_sizes` | $m_1 \dots m_T$ | `np.linspace(0.1, 1.0, 5)`, that is `array([0.1, 0.325, 0.55, 0.775, 1.])` | more points, a smoother curve, linearly more fits | the default five points show the trend; a dense grid such as `np.linspace(0.01, 1.0, 40)` is worth it only when you are reading where the gap closes |
| `cv` | $K$ | `None`, meaning 5-fold | each point averages more fits and jitters less, cost grows linearly, and the largest available $m_T$ grows with $K$ | 5 is fine for a curve, since you are reading a shape rather than ranking candidates |
| `scoring` | $\mathcal{L}$ | `None`, the estimator's own `score` | changes what the axis means entirely | name it explicitly, and use the same measure you will report |
| `shuffle` | none | `False` | randomizes which instances form each prefix instead of taking them in file order | set `True` with a [[Random Seed]] whenever the rows carry any order |
| `exploit_incremental_learning` | none | `False` | warm-starts each larger fit through `partial_fit` instead of refitting, which is faster and yields different fitted models | leave `False` unless the curve is too slow and the estimator supports `partial_fit` |

## Failure modes

- **The prefixes are taken in file order.** `shuffle=False` is the default, and step 3 takes `train[:m_t]`, so on data sorted by the target, by time, or by class the small-size points are fitted on a biased slice and the left half of the curve is meaningless. Stratification does not save you here, because the splitter stratifies the folds and nothing stratifies the prefix.
- **Float sizes are fractions of the fold's training split, not of the dataset.** With $100$ instances at `cv=5` each fold trains on $80$, so `train_sizes=np.linspace(0.01, 1.0, 40)` produces a curve that ends at $80$, not $100$. Integers are read as absolute counts instead, and mixing the two conventions in one array is a silent error.
- **Requested ticks get silently collapsed.** Fractions that round to the same integer are deduplicated, so `train_sizes_abs` comes back shorter than what you passed. Plot against the returned array, never against the array you supplied, or the curve is shifted.
- **Sign.** Every scikit-learn scorer obeys "higher is better", so `scoring="neg_root_mean_squared_error"` returns negative numbers. Plotting the raw arrays draws the picture upside down, and every reading in the table above then inverts.
- **Preprocessing fitted outside the folds.** The estimator handed in has to be the whole [[Pipeline]]. A scaler or imputer fitted once on the full data before the call lets every validation fold contribute to the transformer that is then scored on it, which is [[Data Leakage]], and it flatters the small-size points worst of all, precisely where the curve is being read.
- **Cost.** $K \times T$ fits. The $40$-point dense grid at `cv=5` is $200$ fits of the estimator, and if the estimator is itself a search, the curve is not affordable.

## Implementation

scikit-learn 1.6: `learning_curve` returns three arrays, and five if `return_times=True` adds `fit_times` and `score_times`. The scores come back shaped `(n_ticks, n_folds)`, so averaging across `axis=1` collapses the folds and the leading minus turns the negated scorer back into an error you can plot.

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import learning_curve

train_sizes, train_scores, valid_scores = learning_curve(
    LinearRegression(), X, y,
    train_sizes=np.linspace(0.01, 1.0, 40), cv=5,
    scoring="neg_root_mean_squared_error")

train_errors = -train_scores.mean(axis=1)
valid_errors = -valid_scores.mean(axis=1)
```

Swapping the estimator for a pipeline is the whole of the comparison, and it is why the preprocessing has to be inside the estimator rather than applied to `X` beforehand:

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import PolynomialFeatures

polynomial_regression = make_pipeline(
    PolynomialFeatures(degree=10, include_bias=False),
    LinearRegression())

train_sizes, train_scores, valid_scores = learning_curve(
    polynomial_regression, X, y,
    train_sizes=np.linspace(0.01, 1.0, 40), cv=5,
    scoring="neg_root_mean_squared_error")
```

scikit-learn 1.6: `LearningCurveDisplay` (added in 1.2) wraps the same computation and the plotting together. `negate_score=True` handles the sign that the manual version handles with a minus sign, and `score_name` labels the axis:

```python
from sklearn.model_selection import LearningCurveDisplay

LearningCurveDisplay.from_estimator(
    polynomial_regression, X, y,
    train_sizes=np.linspace(0.01, 1.0, 40), cv=5,
    scoring="neg_root_mean_squared_error",
    negate_score=True, score_name="RMSE")
```

Whichever axis is being swept, the curve feeds [[Model Selection]] from a different direction than a score comparison does. A leaderboard says which of two candidates is ahead today. The curve says whether the next move is to get more data or to change the model at all, and those are not the same question.
