---
note_kind: method
aliases:
  - StandardScaler
  - z-score normalization
  - standard scaling
  - z-score
  - z-scores
  - standardize
  - standardizing
up: "[[Feature Scaling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

Standardization subtracts each column's training mean and divides by its training standard deviation, leaving a column with mean $0$ and variance $1$. It is the default choice for [[Feature Scaling]], and the reason to prefer it over [[Min-Max Scaling]] is that no single extreme value can define the output: an outlier moves $\mu$ and $\sigma$ a little, whereas it moves $x_{\max}$ by its full distance. The cost is that the output is unbounded, so anything downstream that insists on a fixed interval will not get one.

## Algorithm or formula

With $\mu_j$ and $\sigma_j$ computed over the [[Training Set]] only,

$$x'_j = \frac{x_j - \mu_j}{\sigma_j}, \qquad \mu_j = \frac{1}{m}\sum_{i=1}^{m} x^{(i)}_j, \qquad \sigma_j^2 = \frac{1}{m}\sum_{i=1}^{m}\big(x^{(i)}_j - \mu_j\big)^2$$

scikit-learn uses the population divisor $m$, not $m - 1$. The map is affine, so it relocates and rescales the distribution and does nothing else. A skewed column standardizes to a skewed column with mean $0$: standardization does not make anything normal, and treating $z$-scores as if they were normal scores is a common and costly misreading.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `with_mean` | | `True` | turning it on centres the column at $0$, which is what a gradient-based learner wants but which destroys sparsity | off for sparse matrices, where centring is rejected because it would densify the array |
| `with_std` | | `True` | turning it on divides by $\sigma_j$, equalizing spread across columns | off only when you want centring alone, for instance ahead of a method that rescales internally |

`copy` is not in the table: it only decides whether the scaling is done in place or on a fresh array, and the transformed values are the same either way.

## Failure modes

- $\mu$ and $\sigma$ are not robust statistics. A handful of extreme rows inflates $\sigma$, which shrinks every ordinary value toward zero. Standardization is better than min-max here, not immune. `RobustScaler`, which centres on the median and divides by the interquartile range, is the genuinely outlier-resistant option.
- Centring a sparse matrix turns millions of structural zeros into nonzero entries. `StandardScaler` raises on sparse input unless `with_mean=False`.
- A constant column has $\sigma_j = 0$. scikit-learn detects it and sets the scale to $1$ rather than dividing by zero, so the column becomes all zeros and passes through silently.
- Fitting on the full dataset rather than the training split leaks the [[Testing Set]] mean and variance into training, which is [[Data Leakage]]. A [[Pipeline]] is what makes this hard to get wrong.
- A heavy-tailed column is still heavy-tailed afterwards. Fix the shape first with [[Feature Distribution Transformation]], then standardize.

## Implementation

scikit-learn 1.6:

```python
from sklearn.preprocessing import StandardScaler

std_scaler = StandardScaler()
housing_num_std_scaled = std_scaler.fit_transform(housing_num)  # training data
housing_num_test_scaled = std_scaler.transform(housing_num_test)  # stored mu and sigma

std_scaler.mean_, std_scaler.scale_  # the frozen constants
```
