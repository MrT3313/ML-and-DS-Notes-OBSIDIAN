---
note_kind: method
aliases:
  - normalization
  - MinMaxScaler
  - min max scaling
  - min-max normalization
  - minmax scaling
  - rescaling to a range
up: "[[Feature Scaling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Min-max scaling shifts and stretches a column so its training values land exactly on a chosen interval, by default $[0, 1]$. Reach for it over [[Standardization]] when something downstream needs a bounded input: a neural network layer whose activation saturates outside a known range, pixel intensities, or any consumer that would rather have a guaranteed interval than a guaranteed mean. Avoid it when the column has outliers, because the interval is defined by the two most extreme points in the training data.

## Algorithm or formula

With $x_{\min}$ and $x_{\max}$ taken over the [[Training Set]] only,

$$x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}}$$

and for a general target interval $[l, u]$,

$$x'' = x'\,(u - l) + l$$

Both constants are stored at fit time and reapplied unchanged to validation, test and production rows, which is why the scaler belongs inside a [[Pipeline]] rather than in a loose script. Nothing forces a new value into $[l, u]$: a row whose $x$ exceeds the training $x_{\max}$ maps above $u$, which is correct behaviour and the reason `clip` exists.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `feature_range` | $(l, u)$ | $(0, 1)$ | a wider interval spreads the transformed values further apart, and moving $l$ below zero recentres the column | $(0, 1)$ unless the consumer wants zero-centred input, then $(-1, 1)$ |
| `clip` | | `False` | turning it on forces out-of-range test values back onto $[l, u]$, capping them instead of extrapolating | on when a downstream component hard-requires the bound, off when you want to see excursions |

`copy` is not in the table: it only decides whether the scaling is done in place or on a fresh array, and the transformed values are the same either way.

## Failure modes

- One outlier sets the denominator. In a column where almost all districts hold a few thousand people and one holds thirty thousand, every ordinary value is squeezed into the bottom sliver of $[0,1]$ and the differences the model needed are gone. This is the real cost of the method.
- New data outside the training range leaves the interval, so a scaler advertised as producing $[0,1]$ can emit $1.4$ in production. `clip=True` prevents it, but clipped values no longer survive `inverse_transform`.
- A constant column has $x_{\max} - x_{\min} = 0$. scikit-learn's `_handle_zeros_in_scale` replaces the zero range with $1$, so the column silently becomes the constant $l$ rather than raising, and a dead feature passes through unnoticed.
- Fitting on the whole dataset before splitting lets the [[Testing Set]] extremes define the interval, which is [[Data Snooping Bias]]. Min-max is the worst offender here, because the constants are the extremes.
- Sparse input is rejected outright: `MinMaxScaler.transform` does not accept sparse matrices, and subtracting a nonzero $x_{\min}$ would destroy sparsity anyway. Only when $l = 0$ and $x_{\min} = 0$ do zeros stay zero. `MaxAbsScaler`, which divides by $\max |x|$ with no shift, is the sparse-safe analogue.

## Implementation

scikit-learn 1.6:

```python
from sklearn.preprocessing import MinMaxScaler

min_max_scaler = MinMaxScaler(feature_range=(-1, 1))
housing_num_scaled = min_max_scaler.fit_transform(housing_num)  # training data
housing_num_test_scaled = min_max_scaler.transform(housing_num_test)  # stored constants
```
