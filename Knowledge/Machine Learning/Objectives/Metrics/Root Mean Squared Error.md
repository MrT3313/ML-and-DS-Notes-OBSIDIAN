---
note_kind: concept
aliases:
  - RMSE
  - root mean square error
  - root mean squared error
  - root_mean_squared_error
  - neg_root_mean_squared_error
up: "[[Performance Measure]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

Root mean squared error reports the typical magnitude of a model's prediction error, in the units of the target itself: square every residual, average them, take the square root. It is the default number for judging a [[Regression]] model when large mistakes are worse than proportionally large.

## Formal statement

$$\text{RMSE}(\mathbf{X}, h) = \sqrt{\frac{1}{m} \sum_{i=1}^{m} \left( h(\mathbf{x}^{(i)}) - y^{(i)} \right)^{2}}$$

Collect the residuals $r_i = h(\mathbf{x}^{(i)}) - y^{(i)}$ into a vector $\mathbf{r} \in \mathbb{R}^{m}$. Then $\text{RMSE} = \|\mathbf{r}\|_2 / \sqrt{m}$: the $\ell_2$ norm of the residual vector, rescaled so the value does not grow with the size of the evaluation set. Squaring is the whole source of its outlier sensitivity, since a residual twice as large contributes four times as much to the sum before the root flattens the total back.

Mean squared error is $\text{MSE} = \text{RMSE}^{2}$. The square root is strictly increasing on $[0, \infty)$, so $\arg\min_{\boldsymbol\theta} \text{RMSE} = \arg\min_{\boldsymbol\theta} \text{MSE}$: the two pick the same parameters. In practice MSE is what gets optimized, because it is smooth and its gradient is cheaper, and RMSE is what gets reported, because it carries the units of $y$ while MSE carries squared units that mean nothing to a non-specialist. An RMSE of $68{,}000$ on house prices says the typical miss is about $68{,}000$ dollars.

## Where it is used

The [[Performance Measure]] chosen for the [[California Housing]] project, and the scorer passed to [[Cross-Validation]] when ranking candidate models before any of them touch the test set. It is the $p = 2$ member of the [[Lp Norm]] family, and [[Mean Absolute Error]] is the $p = 1$ member; that one difference is the whole argument between them. [[Linear Regression]] ties the knot: its [[Cost Function]] is exactly mean squared error, so least squares is literally minimizing this metric on the training set, which is why a linear model's reported RMSE and its training objective are the same quantity.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import root_mean_squared_error
from sklearn.model_selection import cross_val_score

rmse = root_mean_squared_error(y_train, model.predict(X_train))
rmses = -cross_val_score(model, X_train, y_train,
                         scoring="neg_root_mean_squared_error", cv=10)
```

`root_mean_squared_error` arrived in scikit-learn 1.4. The older route, `mean_squared_error(..., squared=False)`, was deprecated in 1.4 and removed in 1.6, so the `try`/`except ImportError` fallback is only needed on environments older than 1.4. Every scorer follows the convention that higher is better, so error metrics are published negated under a `neg_` name; hence the scoring string `neg_root_mean_squared_error` and the leading minus sign that flips the returned scores back into positive errors.
