---
note_kind: method
aliases:
  - polynomial features
  - PolynomialFeatures
  - polynomial feature expansion
  - polynomial model
  - polynomial fit
up: "[[Linear Regression]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Polynomial regression fits curved data with a straight-line model, by adding powers and products of the existing features as new columns and then running ordinary [[Linear Regression]] on the widened table. No new estimator is involved: the model that gets fitted is the linear one, and the only thing that changed is what it was handed.

Reach for it when a scatter plot or a residual plot shows structure the linear fit is missing, and the relation looks smooth rather than jumpy. It is the cheapest escape from [[Underfitting]] that keeps the interpretability and the closed-form solve intact. Its own risk is the opposite failure: degree is a capacity dial, and turning it up far enough will fit noise, so it comes paired with [[Learning Curve]] to see that happening and with [[Regularization]] as the alternative to turning it back down.

## Algorithm or formula

Fit a transform, then fit a model.

1. Choose a degree $d$. Build the expansion map $\phi$ that sends an instance to every monomial in its features of total degree $1$ through $d$.
2. Apply $\phi$ to every row of the $m \times n$ training matrix $\mathbf{X}$, producing $\boldsymbol\Phi$ of shape $m \times d_{\text{out}}$.
3. Fit linear regression on $(\boldsymbol\Phi, \mathbf{y})$ by the [[Normal Equation]] or by [[Gradient Descent]], unchanged.
4. Predict by applying the same $\phi$ to new instances and taking $\hat{y} = \boldsymbol\theta^{T}\phi(\mathbf{x})$.

### Why powers let a linear model fit a nonlinear problem

This is the whole trick, and it turns on one ambiguity in the word *linear*.

"Linear" in linear regression means **linear in the parameters**, not linear in the inputs. The requirement on the model is that

$$\hat{y} = \boldsymbol\theta^{T}\phi(\mathbf{x}) = \sum_{k} \theta_k \, \phi_k(\mathbf{x})$$

be a linear combination of the parameters $\theta_k$. The basis functions $\phi_k$ may be anything at all, as long as they do not depend on $\boldsymbol\theta$. Powers, products, logs, sines, splines: each is a fixed, known function of $\mathbf{x}$, evaluated before the model ever sees the row. Once evaluated it is just a number in a column, and the model has no way to tell a column that came from $x_1^3$ from a column that was measured directly.

Take the concrete case, one feature and $d = 2$:

$$\hat{y} = \theta_0 + \theta_1 x + \theta_2 x^2$$

Read this as a function of $x$ with $\boldsymbol\theta$ fixed and it is a parabola, plainly nonlinear. Read it as a function of $\boldsymbol\theta$ with $x$ fixed and it is a plane, with coefficients $1$, $x$ and $x^2$, plainly linear. Fitting varies $\boldsymbol\theta$ with the data held fixed, so fitting only ever sees the second reading. The curvature lives entirely inside $\phi$, which is fixed before training and estimates nothing.

**The data shape change, explicitly.** Before expansion the design matrix is

$$\mathbf{X} \in \mathbb{R}^{m \times n}, \qquad \boldsymbol\theta \in \mathbb{R}^{n+1}$$

After expansion it is

$$\boldsymbol\Phi \in \mathbb{R}^{m \times d_{\text{out}}}, \qquad \boldsymbol\theta \in \mathbb{R}^{d_{\text{out}}+1}$$

The row count $m$ never moves. The column count goes from $n$ to $d_{\text{out}}$, and $\boldsymbol\theta$ grows to match. Every column of $\boldsymbol\Phi$ is computed from the row it sits in and nothing else, so no information crosses between instances.

That single substitution, $\mathbf{X} \to \boldsymbol\Phi$, is the entire change, and every formula carries over with the letters swapped:

$$\hat{\boldsymbol\theta} = \big(\boldsymbol\Phi^{T}\boldsymbol\Phi\big)^{-1}\boldsymbol\Phi^{T}\mathbf{y}, \qquad \nabla_{\boldsymbol\theta} J = \frac{2}{m}\,\boldsymbol\Phi^{T}\big(\boldsymbol\Phi\boldsymbol\theta - \mathbf{y}\big)$$

The cost function stays a convex quadratic in $\boldsymbol\theta$, because it was never the inputs that made it quadratic; it was the squaring of the residual. So there is still exactly one minimum, the closed form still lands on it in one solve, and gradient descent still cannot get stuck. Nothing about the optimization got harder. What got harder is generalization, since $\boldsymbol\theta$ now has more entries than the data may be able to pin down, which is the [[Bias-Variance Tradeoff]] moving one notch toward variance.

Seen from outside, this is [[Feature Engineering]] in its purest form: a fixed map $\phi: \mathbb{R}^{n} \to \mathbb{R}^{d_{\text{out}}}$ applied before the hypothesis, so the fitted object is $h \circ \phi$. Polynomial expansion is the case where $\phi$ is chosen mechanically instead of from domain knowledge.

### How many terms, and which

The expansion is not just powers of each feature separately. With more than one feature it includes every cross term too, which is what lets the model express interactions rather than only curvature. On two features $a$ and $b$ at $d = 3$, the seven terms added beyond $a$ and $b$ themselves are

$$a^2,\; ab,\; b^2,\; a^3,\; a^2b,\; ab^2,\; b^3$$

so a product like $ab$ appears alongside the pure powers, and the model can say "the effect of $a$ depends on the level of $b$".

The count is the number of monomials in $n$ variables of total degree at most $d$:

$$d_{\text{out}} = \binom{n + d}{d} \quad \text{with the bias column}, \qquad d_{\text{out}} = \binom{n + d}{d} - 1 \quad \text{without it}$$

Checking that against the example: $n = 2$, $d = 3$ gives $\binom{5}{3} = 10$ with the bias, $9$ without, and $9 - 2 = 7$ beyond the two originals. It matches term for term.

This grows fast enough to be a design constraint rather than a footnote. At $n = 100$ and $d = 2$ the expansion is already $\binom{102}{2} - 1 = 5{,}150$ columns, and since the closed form costs $O(m\,d_{\text{out}}^2)$ the solve is roughly $2{,}600$ times more expensive than the unexpanded one.

`get_feature_names_out()` is how to see the order in scikit-learn 1.6, and `powers_` gives the exponent of each input feature in each output column as an integer array. Do not guess the order from the formula; read it off the transformer.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `degree` | $d$ | `2` | more terms, $\binom{n+d}{d}$ growing steeply, higher capacity, curvier fit, more [[Overfitting]] risk | [[Cross-Validation]] or [[Grid Search]] over $d \in \{1,\dots,6\}$ or so, read against a [[Learning Curve]]; also accepts a `(min_degree, max_degree)` tuple to keep only a band |
| `interaction_only` | | `False` | turning it on **removes** terms: only products of distinct features survive, so $a^2$ and $a^2b$ go and $ab$ stays, dropping the count to $\sum_{i \le d}\binom{n}{i}$ | turn on when interactions are plausible but per-feature curvature is not, or purely to keep the column count survivable at large $n$ |
| `include_bias` | | `True` | turning it on adds one constant column of ones | leave off in front of `LinearRegression`, which fits its own intercept. A duplicate constant is redundant there, and redundant in front of a [[Ridge Regression\|ridge]] fit too as long as `fit_intercept=True`: centering sends the constant column to all zeros before the solve, its coefficient comes back exactly $0$, and the fit is identical to omitting it. Under `fit_intercept=False` it does bite, because the column then carries a real weight and the penalty charges the intercept it stands in for |

Degree is the one that matters; the other two change the shape of the expansion rather than its capacity. Note that `degree` is the total degree of a term, not the per-feature exponent, so $a^2b$ counts as degree $3$.

## Failure modes

- **Degree set too high.** At degree $300$ on a hundred points the curve threads every training point and swings wildly between them; training error goes to nearly zero while validation error explodes. The tell is a [[Learning Curve]] whose two curves never meet and stay far apart as $m$ grows.
- **Expanding before splitting.** `fit_transform` on the whole dataset is not itself a leak, since each output column depends only on its own row, but the [[Standardization]] that almost always follows it is fitted across rows, and fitting that on all the data lets test-set statistics into training. That is [[Data Snooping Bias]], and putting the expansion and the scaler inside a [[Pipeline]] is what prevents it, because cross-validation then refits both on the training folds alone.
- **Unscaled powers.** Raising a column that ranges over $[0, 1000]$ to the fifth power sends it to $[0, 10^{15}]$, while a column in $[0,1]$ stays in $[0,1]$. The expanded matrix becomes badly conditioned and any gradient or penalty term is swamped by the largest column. [[Feature Scaling]] after the expansion, not before, is the fix, since scaling before expansion is undone by the powers.
- **Combinatorial blowup.** $\binom{n+d}{d}$ passes the instance count quickly on wide data, and once $d_{\text{out}} > m$ the design matrix is rank deficient by construction, the fit is exactly determined, and training error hits zero while the model has learned nothing. This is the regime where [[Regularization]] stops being optional.
- **Extrapolation.** A degree-$d$ polynomial diverges like $x^{d}$ outside the range it was fitted on, so predictions just past the edge of the training data are not merely uncertain, they are wrong in a specific and violent direction. A linear fit degrades gracefully there; this does not.

## Implementation

scikit-learn 1.6. The two-step form, which shows what is actually happening:

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

poly_features = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly_features.fit_transform(X)      # (m, n) becomes (m, d_out)

lin_reg = LinearRegression()
lin_reg.fit(X_poly, y)
lin_reg.intercept_, lin_reg.coef_

poly_features.get_feature_names_out()        # which column is which term
poly_features.n_output_features_             # d_out, as counted above
```

The form to actually use, so the expansion and the scaler are refitted inside every fold rather than once over everything:

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

polynomial_regression = make_pipeline(
    PolynomialFeatures(degree=10, include_bias=False),
    StandardScaler(),
    LinearRegression(),
)
polynomial_regression.fit(X, y)
```

Order matters in that pipeline: expand first, scale second. The scaler has to see the powers in order to tame them.
