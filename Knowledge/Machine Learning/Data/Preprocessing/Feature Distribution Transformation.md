---
note_kind: method
aliases:
  - log transform
  - power transform
  - bucketizing
  - bucketize
  - binning
  - discretization
  - RBF
  - radial basis function
  - PowerTransformer
  - KBinsDiscretizer
  - QuantileTransformer
  - distribution transformation
up: "[[Feature Engineering]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## What it does and when

These transforms change the *shape* of a feature's distribution, and run before a scaler, which only changes its range. Reach for one when a histogram shows something a linear model cannot use: a long right tail, several peaks, or a value that matters because it is near some point rather than because it is large. The shape decides which one.

Bucketizing has a second motivation that is not about shape at all: it shrinks what the model has to learn. Fitted on raw income the model is learning a function over an infinite set of possible values; fitted on income cut into lower, middle and upper it is learning three numbers, which is a far easier thing to estimate from few rows. That is the condition under which the operation pays, enough data to pin down a handful of bucket means and not enough to pin down the curve they are standing in for. The standing measurement is Dougherty, Kohavi and Sahami (ICML 1995), who discretized every continuous column in advance across 16 datasets and lifted a naive Bayes classifier from $76.57\%$ to $83.97\%$ average accuracy, past C4.5's own $82.25\%$, with C4.5 itself never significantly worse under the same treatment and significantly better on two datasets. Two things keep that from being a general licence. Their best boundaries are placed using the labels, which is not what any `strategy` below does, and the gain lands on a learner that was assuming a Gaussian per class and was wrong about it. Their reading of why, that a global partition regularizes because it is less exposed to variance estimated from small fragmented data, is offered as a conjecture rather than measured. The operation is not confined to continuous columns either: age arrives already discrete and is still often worth grouping into bands, under 18, 18 to 22, 22 to 30, when the bands are what the question is actually about. Binning maps a feature's values to bucket indices before anything is fitted, and it is not [[Model Quantization]], which maps a fitted model's weights and activations to fewer bits; the shared operation, rounding onto a grid, is the only thing the two have in common.

## Algorithm or formula

**Heavy right tail.** Compress the tail with a concave map: $\log(1 + x)$, $\sqrt{x}$, or $x^{p}$ for $p \in (0,1)$. The point is that a handful of extreme rows stop dominating both the fit and the [[Correlation]] you measured during [[Exploratory Data Analysis]], because a multiplicative gap becomes an additive one. `PowerTransformer` fits the exponent instead of guessing it, using Box-Cox,

$$x^{(\lambda)} = \begin{cases} \dfrac{x^{\lambda} - 1}{\lambda} & \lambda \neq 0 \\[4pt] \log x & \lambda = 0 \end{cases}$$

which requires $x > 0$ strictly, or Yeo-Johnson, the default, which has a separate branch for $x < 0$ and so accepts zero and negative values. `QuantileTransformer` is the brute-force fallback: it replaces each value by its rank and maps that onto a uniform or normal target, fixing any shape at the cost of distorting the spacing between values.

**Several peaks.** Chop the range into roughly equal-sized buckets and replace each value by its bucket index. `KBinsDiscretizer` with `strategy="quantile"` does the equal-sized version. Where the cuts go has to be decided by something, and in practice there are three sources: quantiles of the observed column, which is `strategy="quantile"`; a plain even division of the axis, which is `strategy="uniform"` and is the common-sense choice when equal widths are what the quantity means; and somebody who knows the domain, which corresponds to no `strategy` value at all and means passing the edges yourself through `pandas.cut` or `numpy.digitize` inside a [[Custom Transformer]]. `strategy="kmeans"` belongs to none of the three, reading the boundaries off 1D cluster centres instead of off a convention or a person. The index is a category, not a quantity: left as an integer it asserts that bucket 3 is three times bucket 1, the same ordinality trap as [[Ordinal Encoding]], so follow it with [[One-Hot Encoding]]. `KBinsDiscretizer` does this already, since `encode="onehot"` is its default.

**A meaningful landmark.** Instead of the raw value, give the model its Gaussian similarity to a fixed point $c$:

$$\phi_c(x) = \exp\!\big(-\gamma\,(x - c)^2\big)$$

This peaks at $1$ when $x = c$ and decays both ways, giving the model a feature that says "close to $c$" without building it from a monotonic input. Géron applies it to `housing_median_age` with $c = 35$ and $\gamma = 0.1$, where similarity halves about $\sqrt{\ln 2 / \gamma} \approx 2.6$ years out. The landmark need not be a hand-picked scalar: over several columns at once the same idea reads $\exp\!\big(-\gamma\,\lVert \mathbf{x} - \mathbf{c} \rVert^2\big)$ against a vector centre $\mathbf{c}$, and that centre can be learned rather than chosen, which is what `ClusterSimilarity` does with $k$-means centres ([[Custom Transformer]]).

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| power exponent | $p$ | none, picked by hand | a larger $p$ toward $1$ compresses the tail less; below $1$ compresses more, with $p \to 0$ behaving like $\log$ | start at $\log$ or $\sqrt{\cdot}$, then let `PowerTransformer` fit $\lambda$ |
| `method` | | `yeo-johnson` | Box-Cox is the stricter family and refuses non-positive input | Box-Cox only when every value is strictly positive |
| `standardize` | | `True` | keeping it on returns a zero-mean unit-variance column, folding [[Standardization]] into the step | leave on unless a later scaler already handles it |
| `n_bins` | $K$ | `5` | more bins keep more resolution and produce more one-hot columns, eventually one per instance | cross-validate; tens, not hundreds |
| `strategy` | | `quantile` | `uniform` gives equal widths, `quantile` equal counts, `kmeans` bins by 1D cluster centre | `quantile` for skewed columns, `uniform` when the axis itself is meaningful |
| RBF width | $\gamma$ | none, picked by hand | a larger $\gamma$ narrows the peak, so similarity dies faster away from $c$ | sweep in [[Grid Search]] alongside the landmark $c$ |

## Failure modes

- $\log x$ is undefined at $x = 0$ and negative $x$. Use $\log(1 + x)$ or Yeo-Johnson rather than silently dropping rows.
- A statistic computed on a log-transformed column is a statistic about the logs and not about the column. Averaging the logs and exponentiating returns the geometric mean, $\exp\!\big(\frac{1}{m}\sum_{i} \log x_i\big) = \big(\prod_{i} x_i\big)^{1/m}$, which sits at or below the arithmetic mean and strictly below it unless every value is equal, so it understates exactly the quantity a long-tailed column is usually being summarised for. A confidence interval does not survive the trip back either: exponentiating its endpoints gives an interval asymmetric about the estimate that covers the geometric mean rather than the arithmetic one (Bland and Altman, *BMJ* 312:1079, 1996). Transform for the fit, then read and report on the original scale.
- Fitting the transformer on the full dataset leaks, exactly as a scaler does: quantile boundaries and $\lambda$ are learned parameters, so they belong in a [[Pipeline]] fit on the [[Training Set]] alone ([[Data Leakage]]).
- Binning throws away within-bucket ordering. On a genuinely monotone relationship this is a pure loss, and the extra one-hot columns cost variance for nothing. [[DMLS Ch05 Feature Engineering|DMLS chapter 5]] carries it as the transform least often worth reaching for, which is a judgement rather than a measurement; outside the scarce-data condition above, leaving the column continuous is the honest default.
- Every boundary is a discontinuity the underlying quantity does not have. Two incomes a cent apart on opposite sides of the middle-to-upper cut become different categories and the model is told nothing about how close they were, while two incomes thirty thousand apart inside one bucket become identical. Nothing in the column marks where the cuts belong, which is why the choice falls back on quantiles, an even division, or domain knowledge, and why a boundary placed badly is a cost that no amount of fitting recovers.
- An RBF feature with a badly placed landmark is dead weight. If $\gamma$ is large and $c$ sits where no data lives, every value returns nearly $0$.
- `QuantileTransformer` cannot extrapolate: test values beyond the training range collapse onto the extreme quantiles.
- On tree ensembles the monotonic transforms change nothing at all, since a tree is invariant to them. Only bucketizing and RBF alter what a tree can see, and a [[Skewed Data|skewed]] input costs a tree nothing to begin with.

## Implementation

scikit-learn 1.6:

```python
import numpy as np
from sklearn.preprocessing import FunctionTransformer, PowerTransformer, KBinsDiscretizer
from sklearn.metrics.pairwise import rbf_kernel

log_transformer = FunctionTransformer(np.log, inverse_func=np.exp)
log_population = log_transformer.transform(housing[["population"]])

power = PowerTransformer(method="yeo-johnson", standardize=True)
population_power = power.fit_transform(housing[["population"]])

binner = KBinsDiscretizer(n_bins=10, encode="onehot", strategy="quantile")
income_bins = binner.fit_transform(housing[["median_income"]])

age_simil_35 = rbf_kernel(housing[["housing_median_age"]], [[35]], gamma=0.1)
rbf_transformer = FunctionTransformer(rbf_kernel, kw_args=dict(Y=[[35.0]], gamma=0.1))
```
