---
note_kind: concept
aliases:
  - parameter spaces
  - weight space
up: "[[Model]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The parameter space is the set of every combination of values a [[Model]]'s learnable parameters could take. Training is a search through it for the one point that minimizes the [[Cost Function]], and every optimizer is a different strategy for searching it.

## Formal statement

For a linear model on $n$ features plus a bias term, the parameters are a vector $\boldsymbol\theta = (\theta_0, \theta_1, \dots, \theta_n)$ and the space is

$$\Theta = \mathbb{R}^{n+1}$$

one axis per parameter. The [[Cost Function]] is a scalar field over it, $J : \Theta \to \mathbb{R}$, so the search is a walk on an $(n+1)$-dimensional surface toward $\boldsymbol\theta^{*} = \arg\min_{\boldsymbol\theta \in \Theta} J(\boldsymbol\theta)$. When a model constrains its parameters the space is a subset of $\mathbb{R}^{n+1}$ rather than all of it, which is what a hard constraint does and what [[Regularization]] approximates by bending $J$ instead.

The dimension count is read off the model, not the data:

| model | dimension of $\Theta$ |
|-------|------------------------|
| [[Linear Regression]] or [[Logistic Regression]] on $n$ features | $n + 1$ |
| [[Polynomial Regression]] of degree $d$ on 1 feature | $d + 1$ |
| [[Softmax Regression]] with $K$ classes | $(n + 1)K$ |

Polynomial regression is the case that makes the distinction bite: it is a linear model in a space whose dimension is set by the expanded feature count, not by the original one, which is exactly why it can overfit while still being fitted by the same linear machinery.

### Whether more dimensions means a harder search

Two separate costs hide under "harder", and only one of them scales with dimension.

**Cost per step** does. One [[Batch Gradient Descent]] step over $m$ instances and $n$ features is $O(mn)$, linear in the dimension, so a wider model pays proportionally more arithmetic for every move it makes. The [[Normal Equation]] pays far worse, since solving in closed form costs somewhere between $O(n^{2.4})$ and $O(n^{3})$ in the feature count, which is the whole reason an iterative search is preferred once $n$ is large.

**Number of steps** does not. It is governed not by how many axes there are but by how badly stretched the surface is along them, measured by the condition number of the Hessian,

$$\kappa = \frac{\lambda_{\max}(\mathbf{H})}{\lambda_{\min}(\mathbf{H})}$$

No $n$ appears in it. A well-conditioned problem in a thousand dimensions converges in fewer steps than a badly conditioned one in two, which is why [[Feature Scaling]] is the standard fix for a slow fit and reducing the parameter count is not. The raw intuition that more parameters means a harder search is right about the arithmetic per step and wrong about the number of steps.

Dimension does bite on a cost without [[Convexity]], where more axes means more directions along which the surface can be flat or ambiguous; none of the linear models has such a cost, so it does not arise here.

## Where it is used

[[Gradient Descent]] is the search that runs over it, reading the local slope of $J$ and stepping against it; the size of that step is the [[Learning Rate]] and it is a distance measured in this space. The [[Cost Function]] is the surface defined over it, which is what makes the picture of an elongated bowl meaningful and what [[Feature Scaling]] reshapes. [[Hyperparameter]] draws the boundary that gives the idea its edge: a hyperparameter is not in this space, because it is fixed before the search starts and is never what the gradient moves, so the [[Learning Rate]] and the regularization strength live outside $\Theta$ even though they decide where in $\Theta$ the search lands. Searching over those instead is a different problem with different machinery, namely [[Grid Search]] and [[Randomized Search]], which have no gradient to follow and so enumerate or sample rather than descend.
