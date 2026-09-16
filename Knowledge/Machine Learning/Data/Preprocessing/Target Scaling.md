---
note_kind: method
aliases:
  - label scaling
  - TransformedTargetRegressor
  - target transformation
  - target scaling
  - scaling the labels
  - log target
up: "[[Feature Scaling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Target scaling applies a transformation to the label $y$ rather than to the features, and then undoes it on the way out. Reach for it in a [[Regression]] task when the target spans orders of magnitude or is strongly right-skewed, so that $\log y$ stabilizes the variance and stops the largest labels from owning the [[Cost Function]]. Reach for it also when the model's output geometry expects a particular range, as a neural network with a zero-centred output activation does. The mechanical point is that the fitted model now predicts in the transformed space, and a prediction means nothing until it is mapped back.

## Algorithm or formula

Choose an invertible $g$ and fit $h$ on the transformed labels:

$$z^{(i)} = g\big(y^{(i)}\big), \qquad h = \arg\min \sum_{i=1}^{m} \ell\big(h(\mathbf{x}^{(i)}),\, z^{(i)}\big), \qquad \hat{y} = g^{-1}\big(h(\mathbf{x})\big)$$

For [[Standardization]] of the target, $g(y) = (y - \mu_y)/\sigma_y$ with $\mu_y, \sigma_y$ fit on the training labels only. For a heavy right tail, $g = \log$ and $g^{-1} = \exp$. `TransformedTargetRegressor` is exactly this wrapper: it applies $g$ inside `fit` and $g^{-1}$ inside `predict`, so the inverse can never be forgotten.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `transformer` | $g$ | `None` | supplying a scaler sets the label geometry the model optimizes in | `StandardScaler` for range problems, a log or `PowerTransformer` for skew |
| `func` / `inverse_func` | $g, g^{-1}$ | `None` | supplying a plain function pair replaces `transformer`, which cannot be set at the same time | `func=np.log, inverse_func=np.exp` when no fitted state is needed |
| `check_inverse` | | `True` | keeping it on verifies on a subsample that $g^{-1}(g(y)) \approx y$ and warns if not | turn off only when the round trip is intentionally lossy |

## Failure modes

- Forgetting the inverse. Predictions come back in $z$ space, so a raw `predict` output looks like $-0.3$ where the answer should be \$180,000. `inverse_transform` or `TransformedTargetRegressor` fixes it.
- The reported metric changes meaning. Training on $\log y$ minimizes error in log space, so [[Root Mean Squared Error]] computed there is a relative error on the original scale, not a dollar figure. Score after inverting, or say which space the number lives in.
- Back-transforming the mean. If $\log Y$ is normal then $\exp$ of its mean is the median of $Y$, not the mean: $\mathbb{E}[Y] = \exp(\mu + \sigma^2/2)$. A log-trained regressor systematically under-predicts the mean unless the correction is applied.
- $\log$ is undefined at $y \le 0$. Use $\log(1 + y)$ or Yeo-Johnson when the target can reach zero.
- Fitting the target scaler on all labels before the split leaks [[Testing Set]] information exactly as feature scaling does ([[Data Snooping Bias]]).

## Implementation

scikit-learn 1.6, the explicit form and the wrapper:

```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler

target_scaler = StandardScaler()
scaled_labels = target_scaler.fit_transform(housing_labels.to_frame())

model = LinearRegression().fit(housing[["median_income"]], scaled_labels)
scaled_predictions = model.predict(some_new_data)
predictions = target_scaler.inverse_transform(scaled_predictions)  # required
```

```python
from sklearn.compose import TransformedTargetRegressor

model = TransformedTargetRegressor(LinearRegression(),
                                   transformer=StandardScaler())
model.fit(housing[["median_income"]], housing_labels)
predictions = model.predict(some_new_data)  # already inverted
```

```python
import numpy as np
model = TransformedTargetRegressor(LinearRegression(),
                                   func=np.log, inverse_func=np.exp)
```
