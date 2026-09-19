---
note_kind: index
aliases:
  - Math
up: "[[Home]]"
---

The mathematics the rest of the vault leans on, kept as its own domain so a result can be stated once and cited from every method that uses it instead of being restated in each one. A note earns a place here when something under `Knowledge/Machine Learning/` genuinely depends on it; pure mathematics with no consumer in the vault does not qualify. The domain holds three families of results: the norms two regression metrics both reduce to together with the inequality that makes them norms at all, the moments the distribution-shape statistics are each one order of, and the vector calculus that the iterative training routines are written in. Matrix notation is used across the vault and is defined nowhere in it: [[Normal Equation]]'s closed form $\hat{\boldsymbol{\theta}} = (\mathbf{X}^{T}\mathbf{X})^{-1}\mathbf{X}^{T}\mathbf{y}$ turns on a rank condition that decides whether a least squares fit has one solution or infinitely many, and it asserts that condition rather than defining it.

## Areas

- **Norms and distances.** [[Lp Norm]], the $\ell_1$, $\ell_2$ and $\ell_\infty$ family and the dial between them, and [[Triangle Inequality]], the axiom that fixes where that dial may sit.
- **Distribution shape.** [[Moment]], the raw, central and standardized moments that the mean, the variance, skewness and kurtosis are each one order of.
- **Vector calculus.** [[Partial Derivative]], how a cost responds when one parameter moves and the rest are held still, [[Gradient]], the vector those partials stack into and the direction an optimizer steps against, and [[Convexity]], the property that turns a vanishing gradient from a stationary point into a global minimum.

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
