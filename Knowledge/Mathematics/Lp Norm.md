---
note_kind: concept
aliases:
  - norm
  - norms
  - vector norm
  - p-norm
  - Lp norm
  - Lp Norm
  - Euclidean Norm
  - Euclidean norm
  - Manhattan Norm
  - Manhattan norm
  - taxicab norm
  - L1 norm
  - L2 norm
  - l1 norm
  - l2 norm
  - infinity norm
up: "[[Mathematics]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

A norm assigns one non-negative number to a vector, its length or magnitude, to quantify its size or distance from the origin. The $\ell_p$ family is the standard parameterized way of doing that: a single formula with a dial $p$ that sets how much the largest components of the vector count relative to the rest.

## Formal statement

For $\mathbf{v} \in \mathbb{R}^{n}$ and $p \geq 1$,

$$\|\mathbf{v}\|_p = \left( \sum_{i=1}^{n} |v_i|^{p} \right)^{1/p}$$

The cases that get names:

- $p = 1$, the Manhattan or taxicab norm, $\sum_i |v_i|$, the distance walked on a grid of streets.
- $p = 2$, the Euclidean norm, ordinary straight-line length.
- $p \to \infty$, the largest absolute component. Write $M = \max_i |v_i|$ and factor it out: $\|\mathbf{v}\|_p = M \left( \sum_i (|v_i|/M)^{p} \right)^{1/p}$. Every ratio is at most $1$, so the inner sum stays between $1$ and $n$, and both $1^{1/p}$ and $n^{1/p}$ tend to $1$. The bracket is squeezed to $1$ and $\|\mathbf{v}\|_\infty = M$.
- $p = 0$ is a naming convention, not an instance of the formula. $\|\mathbf{v}\|_0$ counts the nonzero entries, which is $\lim_{p \to 0^{+}} \|\mathbf{v}\|_p^{p} = \lim_{p \to 0^{+}} \sum_i |v_i|^{p}$, not $\lim \|\mathbf{v}\|_p$. It is also not a norm: scaling a vector does not scale the count, so absolute homogeneity fails, $\|\alpha \mathbf{v}\|_0 = \|\mathbf{v}\|_0 \neq |\alpha| \, \|\mathbf{v}\|_0$ for any $\alpha$ outside $\{0, 1, -1\}$. A quasi-norm at best.

Raising $p$ concentrates the value on the biggest component. For $\mathbf{v} = (3, 4)$: $\|\mathbf{v}\|_1 = 7$, $\|\mathbf{v}\|_2 = 5$, $\|\mathbf{v}\|_\infty = 4$, sliding from the full sum down to the single largest entry.

$p \geq 1$ is exactly the condition for the [[Triangle Inequality]] $\|\mathbf{u} + \mathbf{v}\|_p \leq \|\mathbf{u}\|_p + \|\mathbf{v}\|_p$ (Minkowski's inequality) to hold. Below $1$ it fails and the unit ball $\{\mathbf{v} : \|\mathbf{v}\|_p \leq 1\}$ is no longer convex, which is why $p < 1$ penalties turn a fitting problem into a non-convex one.

## Where it is used

[[Root Mean Squared Error]] is the $\ell_2$ norm of the residual vector rescaled by $1/\sqrt{m}$ and [[Mean Absolute Error]] is the $\ell_1$ norm rescaled by $1/m$, so the choice of $p$ is precisely what makes the first sensitive to outliers and the second not. Any [[Cost Function]] built on a norm inherits that behaviour from its $p$. HOML chapter 4 turns the same dial on the parameter vector instead of the residual vector: ridge regression adds an $\ell_2$ penalty on the weights, lasso an $\ell_1$ penalty, and lasso zeroes weights outright because the $\ell_1$ unit ball has corners sitting on the axes.

## Additional Resources
| Title                                                                                   | Link                                               |
| --------------------------------------------------------------------------------------- | -------------------------------------------------- |
| What is Norm in Machine Learning?                                                       | https://www.youtube.com/watch?v=FiSy6zWDfiA        |
| Understanding Vector Norms in Machine Learning (L1 and L2 norms, unit balls, and NumPy) | https://www.youtube.com/watch?v=It2g7sDxdqI        |
| 1.8 Vector Norms \| Linear Algebra Made Easy                                            | https://www.youtube.com/watch?v=DdySKZ9LQ7Q        |
