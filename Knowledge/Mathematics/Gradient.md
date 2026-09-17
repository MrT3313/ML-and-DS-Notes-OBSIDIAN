---
note_kind: concept
aliases:
  - gradient
  - gradients
  - gradient vector
  - nabla
  - del operator
  - grad
  - steepest ascent
up: "[[Mathematics]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The gradient of a scalar function of several variables is the vector holding all of its [[Partial Derivative|partial derivatives]], one per variable. It packages the whole local slope into a single object, and that object points in the direction the function increases fastest.

## Formal statement

For $f : \mathbb{R}^{n+1} \to \mathbb{R}$ and a parameter vector $\boldsymbol\theta = (\theta_0, \dots, \theta_n)^{T}$,

$$\nabla_{\boldsymbol\theta} f(\boldsymbol\theta) =
\begin{bmatrix}
\dfrac{\partial f}{\partial \theta_0}(\boldsymbol\theta) \\[4pt]
\dfrac{\partial f}{\partial \theta_1}(\boldsymbol\theta) \\[2pt]
\vdots \\[2pt]
\dfrac{\partial f}{\partial \theta_n}(\boldsymbol\theta)
\end{bmatrix}$$

The symbol $\nabla$ is nabla, read aloud as "del" or "grad", and the subscript names the variable being differentiated with respect to. Row $j$ is the partial with respect to $\theta_j$, so the index runs $\theta_0$ through $\theta_n$ down the column rather than repeating one $j$.

**Shape.** The gradient has exactly as many entries as $\boldsymbol\theta$ does, and it is written as a column so that it is the same shape as $\boldsymbol\theta$. That is what makes the update rule

$$\boldsymbol\theta^{\text{next step}} = \boldsymbol\theta - \eta \, \nabla_{\boldsymbol\theta} f(\boldsymbol\theta)$$

a legal subtraction of one $(n+1) \times 1$ column from another, with $\eta$ a scalar. A model with $n$ features and a bias searches $n+1$ dimensions of [[Parameter Space]], so its gradient has $n+1$ entries, and every gradient formula can be checked by this test alone before any of the arithmetic is looked at.

For the [[Linear Regression]] cost the whole stack collapses to $\nabla_{\boldsymbol\theta}\text{MSE} = \frac{2}{m}\mathbf{X}^{T}(\mathbf{X}\boldsymbol\theta - \mathbf{y})$; [[Batch Gradient Descent]] is where that product is unpacked. The same collapse works for the [[Logistic Regression]] cost, with a probability error in place of a numeric one, and that note carries the formula. "Features transposed, times residuals" is one shape, which is why one implementation of the step serves both costs.

### Why it points uphill

Among all unit directions, the one that increases $f$ fastest is $\nabla f / \lVert \nabla f \rVert$ and the fastest decrease is its negative, with $\lVert \nabla f \rVert$ the size of that fastest increase. The gradient is therefore the direction of steepest **ascent**, and the minus sign in the update is what turns steepest uphill into steepest downhill. That minus sign is the only reason it is there.

### Zero gradient is necessary, not sufficient

Every interior minimum has $\nabla f(\boldsymbol\theta) = \mathbf{0}$, because a nonzero gradient always offers a direction that lowers $f$; the converse fails, since maxima, saddle points and non-global minima all satisfy it too. [[Convexity]] is what upgrades the condition, and is the entire reason "when the gradient is zero you have converged to a minimum" is safe for the linear models and unsafe in general.

### Size, not just direction

The magnitude $\lVert \nabla f \rVert$ is the usual stopping test, the iteration halted when it falls below a [[Tolerance]]. Entry $j$ has units of cost per unit of $\theta_j$, which is why columns on wildly different scales produce entries on wildly different scales that one [[Learning Rate]] cannot suit at once. [[Partial Derivative]] derives that from the single entry, and it is the argument for [[Feature Scaling]].

## Where it is used

[[Partial Derivative]] supplies the entries, so this note is that one stacked $n+1$ times. [[Gradient Descent]] is the procedure built on the steepest-ascent property, stepping against this vector rather than along it, and [[Batch Gradient Descent]] computes it over the whole training set at every step. [[Convexity]] turns a vanishing gradient from a stationary point into a global minimum. The [[Learning Rate]] $\eta$ is the scalar this vector is multiplied by, converting a direction into a distance travelled, and [[Parameter Space]] is where the vector lives, one axis per parameter. Solving $\nabla_{\boldsymbol\theta}\text{MSE} = \mathbf{0}$ algebraically rather than by steps is the derivation of the [[Normal Equation]], so both routes to fitting a [[Cost Function]] start here.
