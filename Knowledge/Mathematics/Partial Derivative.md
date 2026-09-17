---
note_kind: concept
aliases:
  - partial derivative
  - partial derivatives
  - partials
  - partial
  - partial differentiation
  - partially differentiate
  - first partial derivative
up: "[[Mathematics]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

In a function of several variables, a partial derivative measures how the output changes when exactly one variable is varied and all the others are held constant. In machine learning it is the exact rate of change, the slope, of a [[Cost Function]] with respect to one specific weight or bias in the model.

## Formal statement

For $f : \mathbb{R}^{n+1} \to \mathbb{R}$ and a parameter vector $\boldsymbol\theta$, the partial with respect to $\theta_j$ is the ordinary derivative along that one axis:

$$\frac{\partial f}{\partial \theta_j}(\boldsymbol\theta) = \lim_{h \to 0} \frac{f(\boldsymbol\theta + h \, \mathbf{e}_j) - f(\boldsymbol\theta)}{h}$$

where $\mathbf{e}_j$ is the vector with a $1$ in position $j$ and zeros everywhere else, so $\boldsymbol\theta + h\,\mathbf{e}_j$ moves $\theta_j$ and touches nothing else. The curly $\partial$ rather than a straight $d$ is the whole notational content: it announces that other variables exist and are frozen for the duration.

Freezing them is what makes the computation easy. Every variable but $\theta_j$ is a constant, so the ordinary one-variable rules apply unchanged. One partial therefore answers exactly one question, how much the cost changes if this one weight moves and the rest stay put, and one number per parameter is precisely what [[Gradient Descent]] needs before it can take a step.

**The case this vault runs on.** For the [[Linear Regression]] cost $\text{MSE}(\boldsymbol\theta) = \frac{1}{m}\sum_i (\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)})^{2}$, holding every $\theta_k$ with $k \neq j$ constant leaves only $x_j^{(i)}$ from the inner derivative, and the chain rule on the square brings the exponent down:

$$\frac{\partial}{\partial \theta_j} \text{MSE}(\boldsymbol\theta) = \frac{2}{m} \sum_{i=1}^{m} \big(\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)}\big) \, x_j^{(i)}$$

[[Gradient Descent]] carries the derivation. Two readings are worth keeping here. At $j = 0$, $x_0^{(i)} = 1$, so the bias moves on the mean residual and nothing else. For $j \geq 1$ the residual is weighted by the feature value, which is why a column measured in hundreds of thousands produces a partial hundreds of thousands of times larger at the same quality of fit, and why [[Feature Scaling]] is not cosmetic.

The constant is $2/m$: the $1/m$ is the mean already in the cost and the $2$ is the exponent coming down. Some texts write the cost with $1/(2m)$ precisely so that the $2$ cancels; this vault does not, so the factor stays, and it has to stay in the [[Gradient]] $\frac{2}{m}\mathbf{X}^{T}(\mathbf{X}\boldsymbol\theta - \mathbf{y})$ and in the update rule too. A cost written one way and a gradient written the other is a silent rescaling of the [[Learning Rate]] by a factor of two.

## Where it is used

[[Gradient]] is nothing but the vector these stack into, one entry per parameter, so every property of the gradient is a property of the partials read collectively. [[Batch Gradient Descent]] computes all of them at once over the whole [[Training Set]] as one matrix product. The function being differentiated is always a [[Cost Function]], and which cost it is decides the formula: squared error in [[Linear Regression]], log loss in [[Logistic Regression]]. The $\ell_1$ penalty in [[Lasso Regression]] is the case where the partial fails to exist, and that note carries the repair. Setting every partial to zero produces the [[Normal Equation]], so the closed form is this note's condition solved in one shot rather than approached by steps.
