---
note_kind: method
aliases:
  - elastic net
  - ElasticNet
  - elastic-net
  - elastic net regression
  - elastic net regularization
  - ElasticNetCV
up: "[[Linear Regression]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Elastic net is [[Linear Regression]] with a weighted sum of the $\ell_1$ and $\ell_2$ terms added to the training [[Cost Function]], rather than one or the other, so it sits on a line between [[Lasso Regression]] and [[Ridge Regression]] with a mixing ratio $r$ deciding where. At $r = 0$ it is exactly ridge, at $r = 1$ exactly lasso, and in between it is charging two [[Lp Norm|norms]] of the weight vector at once. It is one of the three penalized linear models, distinguished from the other two by having a second hyperparameter that says how the penalty is split, and as in those two the [[Regularization]] penalty applies during training only.

What the $\ell_2$ half buys is stability of the selection the $\ell_1$ half makes. The $\ell_1$ term is indifferent between putting a weight of $6$ on one of two near-duplicate features and splitting it $3$ and $3$, because $\lvert 6 \rvert = \lvert 3 \rvert + \lvert 3 \rvert$. The $\ell_2$ term is not indifferent, for the reason set out in [[Ridge Regression]], and it breaks the tie in favour of the split. That tie-break is what [[Regularization]] means when it prefers elastic net to lasso on correlated features and on $n > m$ data, and [[Lasso Regression]] shows both cases with numbers.

The cost is a second hyperparameter, so $r$ has to be earned rather than assumed.

## Algorithm or formula

Take the two sibling penalties and mix them:

$$J(\boldsymbol\theta) = \text{MSE}(\boldsymbol\theta) \;+\; r\left( 2\alpha \sum_{i=1}^{n} \lvert \theta_i \rvert \right) \;+\; (1 - r)\left( \frac{\alpha}{m} \sum_{i=1}^{n} \theta_i^{2} \right)$$

Written this way the endpoint claim is exact rather than approximate: substituting $r = 1$ leaves the lasso cost as [[Lasso Regression]] states it, and $r = 0$ leaves the ridge cost as [[Ridge Regression]] states it, term for term. As in both siblings the sum starts at $i = 1$ and the bias $\theta_0$ is not penalized.

$\alpha$ still sets how much total penalty there is; $r$ only decides how it is divided. The two interact, because moving weight onto the $\ell_1$ half at fixed $\alpha$ makes the fit sparser as well as differently shrunk, so they are tuned jointly rather than one after the other.

### The library form and what carries across

`ElasticNet(alpha=a, l1_ratio=rho)` minimizes

$$\frac{1}{2m}\lVert \mathbf{y} - \mathbf{X}\mathbf{w} \rVert_2^2 \;+\; a \rho \lVert \mathbf{w} \rVert_1 \;+\; \frac{a(1 - \rho)}{2} \lVert \mathbf{w} \rVert_2^2$$

`l1_ratio` is $r$: scikit-learn documents `l1_ratio=0` as a pure $\ell_2$ penalty and `l1_ratio=1` as a pure $\ell_1$ penalty, the same direction as the cost function above. That much transfers.

The value of `alpha` does not transfer at both ends, and this is the trap. Doubling the library objective gives $\text{MSE} + 2a\rho\lVert\mathbf{w}\rVert_1 + a(1-\rho)\lVert\mathbf{w}\rVert_2^2$. The $\ell_1$ terms then match the cost function above exactly, but the $\ell_2$ terms differ by a factor of $m$, because `ElasticNet` averages its data term over the instances while `Ridge` sums it. Both endpoints check out on $m = 100$ points:

| call | equals | fitted $(\theta_0, \theta_1)$ |
|---|---|---|
| `ElasticNet(alpha=0.1, l1_ratio=1.0)` | `Lasso(alpha=0.1)` | $\theta_1 = 0.27101065$ from both |
| `ElasticNet(alpha=0.1, l1_ratio=0.0)` | `Ridge(alpha=10.0)`, that is $\alpha \cdot m$ | $(1.20655238, 0.35305465)$ from both |

So an `alpha` tuned on `Lasso` can be handed straight to `ElasticNet`, and an `alpha` tuned on `Ridge` has to be divided by $m$ first. The full convention table is in [[Ridge Regression]].

At any $r$ strictly between $0$ and $1$ no single $\alpha$ reconciles the two, because the halves need $\alpha = a$ and $\alpha = am$ respectively; read the library form as authoritative in the interior and use the cost function above only to see what the endpoints recover.

Like [[Lasso Regression]], elastic net has no closed form: the $\ell_1$ half is not differentiable at zero. `ElasticNet` is fitted by the same coordinate descent solver, where the $\ell_2$ half changes only the denominator of each coordinate update while the $\ell_1$ half supplies the soft threshold, which is why exact zeros survive the mixture.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| penalty strength | $\alpha$ | `1.0` | more total shrinkage on both halves at once, so a sparser and flatter model, moving from [[Overfitting]] toward [[Underfitting]] | log grid jointly with $r$ under [[Cross-Validation]]. `ElasticNetCV` builds a path of `n_alphas=100` values per `l1_ratio` spanning `eps=1e-3` of the range |
| mixing ratio | $r$, `l1_ratio` | `0.5` | more of the penalty is $\ell_1$, so more weights land on exactly zero and correlated features stop sharing their weight. At `1.0` it is [[Lasso Regression]], at `0.0` it is [[Ridge Regression]] | pass `ElasticNetCV` a list rather than a float. The documented heuristic is to crowd the candidates near $1$, `[.1, .5, .7, .9, .95, .99, 1]`, because the interesting behaviour changes fast at the lasso end and slowly at the ridge end |
| iteration cap | `max_iter` | `1000` | coordinate descent runs longer and gets closer to the minimum, which changes which features survive and not only their size | raise it whenever a `ConvergenceWarning` appears; the default is not enough on collinear or badly scaled data |
| stopping tolerance | `tol` | `1e-4` | stops on a smaller improvement, so an earlier and less exact solution and a less trustworthy set of nonzero weights | tighten alongside `max_iter` when repeated fits disagree about the support |
| fit intercept | `fit_intercept` | `True` | `False` forces $\theta_0 = 0$, a different model unless $\mathbf{X}$ and $\mathbf{y}$ are already centered | leave `True` |
| non-negativity | `positive` | `False` | `True` constrains every weight to be at least zero, a genuine restriction on the solution | set it only when a negative weight is meaningless in the domain |
| coordinate order | `selection` | `"cyclic"` | not ordered. `"random"` updates a randomly chosen coordinate each iteration, usually converging faster and, since convergence is approximate, sometimes landing somewhere slightly different | leave `"cyclic"` unless convergence is slow; if you switch, pin `random_state` |

## Failure modes

- **Tuning $\alpha$ first and $r$ afterwards finds the wrong pair.** The two are not separable: changing $r$ redistributes the penalty and therefore changes which $\alpha$ is right, so a grid over $\alpha$ at fixed $r$ followed by a grid over $r$ at that $\alpha$ can miss the good region entirely. Search them together, which is what `ElasticNetCV` with a list of `l1_ratio` values does.
- **`l1_ratio=0` is the wrong tool for the job it names.** It is legal and mathematically equals ridge, but the coordinate descent solver is built around a nonzero threshold and handles it badly. `ElasticNet(alpha=0.1, l1_ratio=0.0)` raises a `ConvergenceWarning` that ends by telling you models with no $\ell_1$ term are fitted more efficiently by `Ridge` or `RidgeCV`. Take the advice; do not sit at the ridge endpoint of the elastic net.
- **Unscaled features corrupt both halves at once.** One $\alpha$ charges every weight at one rate under either norm, so an unscaled column is crushed by the $\ell_2$ half or zeroed by the $\ell_1$ half; put [[Standardization]] in front of the model inside a [[Pipeline]], see the numbers in [[Ridge Regression]] and the mechanism in [[Feature Scaling]].
- **Sparsity bought by mixing is not the same sparsity lasso reports.** Below $r = 1$ the $\ell_2$ half keeps correlated features in the model together, so the count of surviving features is larger and any one of them can be near-duplicated by another. That is the right answer for prediction and the wrong answer if you wanted a minimal set of features to go and measure.
- **Two more hyperparameters than plain least squares, on a model that may not need them.** If nothing is correlated and $m \gg n$, the mixture costs a full extra dimension of search and returns a model neither sibling would have missed. Justify $r$ with [[Cross-Validation]] rather than reaching for it by habit.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import ElasticNet

elastic_net = ElasticNet(alpha=0.1, l1_ratio=0.5)
elastic_net.fit(X, y)
elastic_net.predict([[1.5]])
```

scikit-learn 1.6, how you should actually fit it: scale inside a pipeline and search $\alpha$ and $r$ together.

```python
from sklearn.linear_model import ElasticNetCV
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    StandardScaler(),
    ElasticNetCV(l1_ratio=[.1, .5, .7, .9, .95, .99, 1],
                 cv=5, max_iter=100_000, random_state=42),
)
model.fit(X_train, y_train)
model[-1].alpha_, model[-1].l1_ratio_
```

scikit-learn 1.6, the same penalty fitted by stochastic gradient descent, for a training set too large to hold. Note that `SGDRegressor` normalizes its data term by $m$ and so wants a different `alpha` than `Ridge` does:

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(penalty="elasticnet", alpha=0.001, l1_ratio=0.5,
                       max_iter=1000, random_state=42)
sgd_reg.fit(X_train, y_train.ravel())
```
