---
note_kind: index
aliases:
  - Math
up: "[[Home]]"
---

The mathematics the rest of the vault leans on, kept as its own domain so a result can be stated once and cited from every method that uses it instead of being restated in each one. A note earns a place here when something under `Knowledge/Machine Learning/` genuinely depends on it; pure mathematics with no consumer in the vault does not qualify. Today the domain holds two families of results, the norms two regression metrics both reduce to together with the inequality that makes them norms at all, and the moments the distribution-shape statistics are each one order of.

## Areas

- **Norms and distances.** [[Lp Norm]], the $\ell_1$, $\ell_2$ and $\ell_\infty$ family and the dial between them, and [[Triangle Inequality]], the axiom that fixes where that dial may sit.
- **Distribution shape.** [[Moment]], the raw, central and standardized moments that the mean, the variance, skewness and kurtosis are each one order of.

## What is missing

Nearly everything. The honest list, in the order the book will force it:

- Linear algebra: transpose, inverse, rank, and the normal equation that HOML chapter 4 uses to solve linear regression in closed form. Every formula in the vault already written in matrix notation is borrowing against this.
- Vector calculus: partial derivatives, the gradient, and convexity in general, all needed before gradient descent in chapter 4 can be described as anything more than a metaphor about walking downhill. Convexity currently exists here only for norms, as a consequence of the triangle inequality, with no general definition behind it.
- Statistics, which chapter 2 already opened a gap in: Pearson's correlation coefficient behind [[Correlation]] and the bootstrap used to put a confidence interval around a test-set error. Both are currently used as black boxes. The moment definition of skewness behind [[Skewed Data]] was the third of these and is now covered by [[Moment]].
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
