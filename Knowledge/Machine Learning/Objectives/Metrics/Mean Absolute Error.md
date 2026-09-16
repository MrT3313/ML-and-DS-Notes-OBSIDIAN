---
note_kind: concept
aliases:
  - MAE
  - average absolute deviation
  - mean absolute error
  - mean absolute deviation
  - L1 loss
  - mean_absolute_error
  - neg_mean_absolute_error
up: "[[Performance Measure]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

Mean absolute error reports the average size of a prediction error with no squaring, so a miss of ten counts as ten times a miss of one rather than a hundred times. It is the robust alternative to [[Root Mean Squared Error]] for a [[Regression]] model.

## Formal statement

$$\text{MAE}(\mathbf{X}, h) = \frac{1}{m} \sum_{i=1}^{m} \left| h(\mathbf{x}^{(i)}) - y^{(i)} \right|$$

With the residuals gathered into $\mathbf{r} \in \mathbb{R}^{m}$, this is $\|\mathbf{r}\|_1 / m$, the $\ell_1$ norm of the residual vector averaged over the instances. Like RMSE it carries the units of the target.

The absolute value has a corner at $r_i = 0$, where the derivative jumps from $-1$ to $+1$ and only a subgradient in $[-1, 1]$ exists. Gradient-based optimizers therefore behave worse on it than on squared error, which is one reason squared error stays the training objective even in projects that report MAE.

## VS

Against RMSE, the structural difference is not "more or less sensitive to outliers" but which summary of the target each one estimates. The constant $c$ minimizing $\mathbb{E}\left[(Y - c)^{2}\right]$ is the mean $\mathbb{E}[Y]$; the constant minimizing $\mathbb{E}\left[|Y - c|\right]$ is a median. Conditioning on the input, a model fit to squared error estimates the conditional mean $\mathbb{E}[Y \mid \mathbf{x}]$ and a model fit to absolute error estimates the conditional median. A single extreme label drags a mean and leaves a median where it was, and that is the actual mechanism behind the robustness claim.

Géron's rule of thumb follows from it: prefer RMSE when outliers are exponentially rare, so the error distribution is roughly bell shaped and the mean is a fair summary, and prefer MAE when the dataset has many outliers and the mean stops describing anything.

## Where it is used

One of the two candidate [[Performance Measure]] choices for the chapter's housing project, evaluated the same way and swapped into [[Cross-Validation]] by changing one scoring string. It is the $p = 1$ member of the [[Lp Norm]] family, which is why it behaves like a taxicab distance over residuals rather than a straight-line one. Used as a training objective rather than a report it becomes a [[Cost Function]], sometimes called L1 loss, and gives a model that predicts conditional medians.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import mean_absolute_error

mae = mean_absolute_error(y_train, model.predict(X_train))
```

The cross-validation scoring string is `neg_mean_absolute_error`, negated for the same higher-is-better convention that gives RMSE its `neg_` prefix.
