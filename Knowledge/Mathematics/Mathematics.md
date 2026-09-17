---
note_kind: index
aliases:
  - Math
up: "[[Home]]"
---

The mathematics the rest of the vault leans on, kept as its own domain so a result can be stated once and cited from every method that uses it instead of being restated in each one. A note earns a place here when something under `Knowledge/Machine Learning/` genuinely depends on it; pure mathematics with no consumer in the vault does not qualify. The domain holds three families of results: the norms two regression metrics both reduce to together with the inequality that makes them norms at all, the moments the distribution-shape statistics are each one order of, and the vector calculus that the iterative training routines are written in.

## Areas

- **Norms and distances.** [[Lp Norm]], the $\ell_1$, $\ell_2$ and $\ell_\infty$ family and the dial between them, and [[Triangle Inequality]], the axiom that fixes where that dial may sit.
- **Distribution shape.** [[Moment]], the raw, central and standardized moments that the mean, the variance, skewness and kurtosis are each one order of.
- **Vector calculus.** [[Partial Derivative]], how a cost responds when one parameter moves and the rest are held still, [[Gradient]], the vector those partials stack into and the direction an optimizer steps against, and [[Convexity]], the property that turns a vanishing gradient from a stationary point into a global minimum.

## What is missing

Nearly everything. The honest list, in the order the book will force it:

- Linear algebra: transpose, inverse and rank. What a transpose does, what makes $\mathbf{X}^{T}\mathbf{X}$ invertible or not, what rank is, and what the pseudoinverse substitutes when the inverse fails all go undefined in this domain. Every formula in the vault written in matrix notation borrows against that, and [[Normal Equation]] is where the borrowing shows most plainly: its closed form $\hat{\boldsymbol{\theta}} = (\mathbf{X}^{T}\mathbf{X})^{-1}\mathbf{X}^{T}\mathbf{y}$ turns on a rank condition that decides whether a least squares fit has one solution or infinitely many, and that condition is asserted rather than defined anywhere.
- Vector calculus beyond the first order: the Hessian as an object in its own right, which [[Convexity]] uses only as a test, and the second-order methods that step with it rather than merely checking it. The chain rule in the multivariable form is the other gap, needed before backpropagation in the neural network chapters is anything but a black box.
- Statistics: Pearson's correlation coefficient behind [[Correlation]] and the bootstrap used to put a confidence interval around a test-set error. Both are used as black boxes. The moment definition of skewness that [[Skewed Data]] rests on is the one piece of this the domain does define, in [[Moment]].
- Probability: distributions, expectation, and variance are assumed by every note that says "expected value" and are defined by none of them.

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
