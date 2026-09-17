---
note_kind: concept
aliases:
  - convex
  - convexity
  - convex function
  - strictly convex
  - non-convex
  - nonconvex
  - convex optimization
up: "[[Mathematics]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

A function is convex when its graph never rises above the chord joining any two points on it, so it curves one way only and has no dip that is not the lowest dip.

## Formal statement

$f$ is convex when, for every $\mathbf{u}, \mathbf{v}$ in its domain and every $\lambda \in [0,1]$,

$$f\big(\lambda\mathbf{u} + (1-\lambda)\mathbf{v}\big) \leq \lambda f(\mathbf{u}) + (1-\lambda) f(\mathbf{v})$$

As $\lambda$ runs from $0$ to $1$ the left argument traces the segment from $\mathbf{v}$ to $\mathbf{u}$, so the left side is the function along that segment and the right side is the chord above it.

### What convexity buys

**Every local minimum is global.** A point lower than a claimed local minimum would, by the chord inequality, put points arbitrarily close to that minimum lower still, which contradicts it being local. So on a convex function "a minimum" and "the minimum value" are the same thing, and where an optimizer was initialized cannot change the value it converges to.

**A vanishing gradient becomes sufficient.** Without convexity a zero [[Gradient]] says only "stationary", which a maximum and a saddle point both satisfy. A differentiable convex function lies above every one of its tangent planes,

$$f(\mathbf{z}) \geq f(\boldsymbol\theta) + \nabla f(\boldsymbol\theta)^{T}(\mathbf{z} - \boldsymbol\theta) \quad \text{for every } \mathbf{z}$$

so setting $\nabla f(\boldsymbol\theta) = \mathbf{0}$ leaves $f(\mathbf{z}) \geq f(\boldsymbol\theta)$ everywhere: a stationary point is the global minimum, which is what [[Gradient Descent]] stops on and what [[Normal Equation]] solves for directly.

**A second derivative that never goes negative is the working test.** For a twice differentiable $f$, convexity is equivalent to the Hessian $\nabla^{2} f(\boldsymbol\theta)$ being positive semidefinite everywhere, which in one variable is just $f'' \geq 0$ and in several is $\mathbf{v}^{T}\nabla^{2} f\,\mathbf{v} \geq 0$ for every direction $\mathbf{v}$. This is how convexity is actually checked in practice, the chord inequality being awkward to verify directly. Make the inequality strict for every $\mathbf{v} \neq \mathbf{0}$ and the function is **strictly convex**, which upgrades "the minimum value is unique" to "the minimizing point is unique": a convex function can have a flat trough of minimizers, a strictly convex one cannot.

**Convexity survives the operations a cost is built from.** A non-negative weighted sum of convex functions is convex, and a norm is convex because subadditivity and homogeneity give the chord inequality, as [[Triangle Inequality]] shows and [[Lp Norm]] uses. So a convex loss plus an $\ell_1$ or $\ell_2$ penalty stays convex, in [[Ridge Regression]], [[Lasso Regression]] and [[Elastic Net Regression]] alike.

**The two costs behind the linear models are both convex.** The [[Linear Regression]] cost is a quadratic whose Hessian does not depend on $\boldsymbol\theta$ at all and is positive semidefinite for every possible design matrix, which is why the bowl is the same shape whatever the data; [[Normal Equation]] carries that derivation, and the rank condition that decides whether the bottom is a point or a flat subspace. It is strictly convex exactly when $\mathbf{X}$ has full column rank, which is the same rank condition read from the other side. The [[Logistic Regression]] cost is convex too, for a Hessian that varies with $\boldsymbol\theta$ but never goes negative, and that note carries it; it is not strictly convex in general, and on separable data its infimum is approached without ever being attained, so convexity does not by itself promise a minimizer exists. That second case is where convexity earns the most, because there is no closed form to fall back on: setting the gradient to zero gives an equation with $\boldsymbol\theta$ inside a sigmoid as well as outside it, which no rearrangement isolates. No closed form, but no spurious minima either.

### What convexity does not promise

Convexity is a statement about the shape of the [[Cost Function]], not about any particular run of [[Gradient Descent]]: the step size still has to sit under the stability ceiling and the run still has to be long enough. It removes one failure mode, not tuning.

## Where it is used

[[Gradient Descent]] rests its central guarantee here, and [[Batch Gradient Descent]] converges to a point rather than a neighbourhood on such a cost, the [[Learning Rate]] still bounded above on it. [[Linear Regression]] and [[Logistic Regression]] are the two instances worked out here.
