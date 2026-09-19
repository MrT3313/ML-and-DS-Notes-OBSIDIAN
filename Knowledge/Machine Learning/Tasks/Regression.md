---
note_kind: index
aliases:
  - regressor
  - regressors
  - regression task
up: "[[Machine Learning]]"
---

Every method that predicts a continuous numerical value for an input: a house price, a temperature, next month's sales. A [[Supervised Learning]] task. Géron's terms run along two independent axes. Univariate versus multivariate counts the outputs, one predicted value per instance or several at once; simple versus multiple counts the inputs, one feature or many. The housing problem in [[HOML Ch02 End-to-End Machine Learning Project|HOML chapter 2]] is both multiple and univariate: many features per district, a single median house value out.

## Methods

- [[Linear Regression]]: weighted sum of features, least squares. The baseline, and the worked example on [[California Housing]].
Fitting routes rather than models, listed because which one you reach for changes what is feasible rather than what is fitted:

- [[Normal Equation]]: the closed form for the one above, reaching the least-squares optimum in a single matrix solve. The route to take when the feature count is modest and every row is in memory.
- [[Gradient Descent]]: the iterative alternative, and the only route once $n$ is large or the closed form does not exist. [[Batch Gradient Descent]], [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] are its three variants, told apart by how many instances one gradient is computed on; `SGDRegressor` is the second of them wrapped as a regressor.
- [[Polynomial Regression]]: the same linear estimator handed powers and products of its own columns, so the fitted curve bends while the model stays linear in its parameters. Degree is the capacity dial.
- [[Ridge Regression]]: linear regression with an $\ell_2$ penalty on the weights during training. Shrinks every weight toward zero, drops none of them, and is the default when least squares has more freedom than the data justifies.
- [[Lasso Regression]]: the same with an $\ell_1$ penalty, which lands weights on exactly zero. A regressor that selects its own features, at some cost in accuracy.
- [[Elastic Net Regression]]: both penalties at once, mixed by a ratio. The repair for lasso's arbitrary choice among correlated features.

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```
