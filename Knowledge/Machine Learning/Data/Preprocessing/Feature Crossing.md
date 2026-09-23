---
note_kind: method
aliases:
  - feature crossing
  - feature cross
  - feature crosses
  - crossed feature
  - crossed features
  - cross feature
  - feature interaction
  - interaction feature
  - interaction term
  - interaction terms
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

Crossing combines two or more existing features into one new feature: a product of two numbers, or a joint category built from two category columns. You build one to hand the model an *interaction*, the situation where the effect of one feature depends on the value of another, because some model families cannot express that on their own and will never discover it from the parent columns alone.

Which families, and why, is the part worth getting right, because the two common weaknesses are different weaknesses and only one of them is the one crossing repairs.

**[[Linear Regression|Linear]] and [[Logistic Regression|logistic]] regression are additive in their inputs.** The score is $\boldsymbol\theta^{T}\mathbf{x} = \theta_0 + \sum_j \theta_j x_j$, so the partial effect of $x_j$ is $\theta_j$ no matter what every other feature is doing. There is no coefficient in that expression whose value could depend on a second feature, so an interaction is not expressible at all, at any setting of $\boldsymbol\theta$, unless a column carrying it is handed in from outside. Crossing is what hands it in.

**A decision tree is the opposite case and expresses interactions natively.** A split tests $x_j \le t$ ([[Feature Scaling]]), and the region reached at a leaf is the intersection of every test on the path from the root, so a path `x1 <= a` then `x2 <= b` picks out exactly the rows where both hold. That conjunction *is* an interaction term, and the tree built it without being given one. Depth is the only bound: a tree of depth $d$ conjoins at most $d$ conditions, so it reaches interactions of order up to $d$ and no further. What a tree is genuinely bad at is something else entirely. Every split is perpendicular to one axis, so every leaf region is an axis-aligned box, and a smooth or diagonal boundary such as $x_1 + x_2 = c$ comes back as a staircase of boxes that more splits refine but never straighten. Crossing does nothing about that second weakness, and it is redundant against the first, which is why a cross rarely pays on a tree. The one exception is the greedy search rather than the representation: on a pure XOR-shaped target neither parent shows any impurity gain at the root, so the split that would start the conjunction is never chosen, and the crossed column is how you get the interaction past the greedy step.

Crossing matters less for a neural network, which composes nonlinearities across hidden layers and so can represent an interaction without being handed one, though that is a claim about what the architecture can express and not a finding that hand-built crosses have stopped paying, since production recommenders that keep an explicit crossed linear term beside a deep one are the standing counterexample.

One more distinction, because the word *nonlinear* covers two things and a cross buys only one of them. A cross buys interaction. It buys no curvature at all: with $x_2$ held fixed the fitted function is still a straight line in $x_1$. Curvature in a single feature comes from a power, a bin or a basis function instead ([[Feature Distribution Transformation]]).

## Algorithm or formula

**Categorical parents.** Let features $1$ through $k$ have level sets $C_1, \dots, C_k$. Their cross is a new categorical feature whose level set is the Cartesian product $C_1 \times \cdots \times C_k$, so its cardinality is

$$\Big| \prod_{j=1}^{k} C_j \Big| = \prod_{j=1}^{k} |C_j|$$

The overfitting risk is a consequence of that product rather than a separate warning sitting beside it. The parents contribute $\sum_j |C_j|$ levels between them; the cross contributes $\prod_j |C_j|$. With $m$ training instances the average level of the cross is supported by

$$\frac{m}{\prod_{j=1}^{k} |C_j|}$$

rows, and that number falls multiplicatively with every parent added while $m$ stays where it was. Each level of the cross gets its own parameter, and a parameter estimated from a handful of rows is fitted to the noise in those rows, because there is nothing else in them to fit. The model then scores well on the training set and badly on new data, which is [[Overfitting]] arrived at by construction rather than by bad luck. The same product is what makes [[One-Hot Encoding]] of a cross explode in width, since encoding the cross costs $\prod_j |C_j|$ columns where encoding the parents separately costs $\sum_j |C_j|$. And when one parent's level set is not fixed in advance, ex. an item identifier that gains members in production, the cross inherits that and its level set is unbounded too, which is the situation [[Feature Hashing]] answers by fixing the output width instead of the input vocabulary.

**Numeric parents.** The same operation is the product $x_i x_j$, the pairwise interaction term. What the extra column buys is visible in the partial derivative. Fit

$$\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \theta_3 x_1 x_2, \qquad \frac{\partial \hat{y}}{\partial x_1} = \theta_1 + \theta_3 x_2$$

and the slope along $x_1$ now depends on the level of $x_2$, which is exactly the statement the additive model had no way to make. Taking every distinct pair at once, a table of $n$ numeric features becomes

$$n + \binom{n}{2} = n + \frac{n(n-1)}{2}$$

columns, which is what `PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)` returns; at order $k$ it is $\sum_{i=1}^{k}\binom{n}{i}$. Stepping past distinct pairs to every monomial up to a degree, pure powers included, is [[Polynomial Regression]], which is this operation specialized to numeric columns and chosen mechanically rather than one product at a time, and which carries the count and the fit for that case.

**The two cases are one case.** Once the parents are one-hot encoded, a product of indicator columns is the indicator of a joint level,

$$\mathbb{1}[x_1 = a] \cdot \mathbb{1}[x_2 = b] = \mathbb{1}\big[(x_1, x_2) = (a, b)\big]$$

so multiplying out the encoded columns gives precisely the one-hot encoding of the cross. They differ only in what grows: crossing categoricals grows a level set, crossing numerics grows a column count.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| which features to cross | $\{j_1, \dots, j_k\}$ | none, chosen by hand | every parent added multiplies the level count or the column count and divides the rows supporting each level | start from a domain hypothesis about what depends on what, keep only the crosses that move a [[Cross-Validation]] score |
| `degree` | $k$ | `2` | the order of the cross, taking $k$-way terms and producing $\sum_{i \le k}\binom{n}{i}$ columns | stay at $2$ unless a three-way effect can be argued for; the support per level makes the argument expensive |
| `interaction_only` | | `False` | turning it on **removes** terms, keeping only products of distinct features, so degree $2$ gives $n + \binom{n}{2}$ columns instead of $n + \binom{n+1}{2}$ | on when you want interaction and not curvature, which is the crossing case |

`include_bias` is pinned rather than tuned: it adds one constant column, and it stays `False` in front of any estimator that fits its own intercept. The settings that decide what happens to a level of the cross unseen at fit time, `handle_unknown` and `min_frequency`, belong to the encoder downstream and are tabulated in [[One-Hot Encoding]].

## Failure modes

- **High-cardinality parents.** Two columns of $1{,}000$ levels each cross to $1{,}000{,}000$ levels, so on a million training rows the average crossed level holds a single instance and every coefficient is a memorized row. The cardinality of the parents is the thing to check before building the cross, not after.
- **Nearly collinear parents.** If $x_2$ is close to a monotone function of $x_1$, then $x_1 x_2$ is close to $x_1^2$ and carries almost nothing the parents did not already carry. The column is not free: it worsens the conditioning of the solve and splits one effect across three coefficients that are individually meaningless.
- **Crossing before splitting.** The level set of a categorical cross is learned across rows, so building it on the full dataset before the split lets the test split's levels into the fitted vocabulary, which is [[Data Leakage]]. Done correctly, inside a [[Pipeline]], the sparsity shows up instead as its honest symptom: crossed levels that appear only in the test split and arrive at the encoder as unknowns.
- **A parent whose level set is unbounded.** Crossing a fixed feature with a column that keeps gaining members in production makes the cross unbounded too, so any encoder that memorizes a vocabulary at fit time is stale the day after deployment and every new combination is either an error or an all-zero row.
- **Crossing a tree.** Added to a tree ensemble, a crossed column is usually redundant against a conjunction of splits the tree can already form, and it gives the greedy split search one more candidate correlated with its own parents, which spreads impurity-based importances across the three and makes the ranking harder to read for no gain in fit.
- **Reaching for a cross to fix curvature.** A product of two features leaves the fit a straight line along each axis with the other held fixed, so a single feature entering nonlinearly is untouched by any number of crosses.

## Implementation

scikit-learn 1.6, with NumPy 2.x and pandas 2.x. The numeric case is one transformer, and the two arguments that make it a cross rather than a polynomial expansion are `interaction_only=True`, which drops the pure powers, and `include_bias=False`, which drops the constant column:

```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import PolynomialFeatures, OneHotEncoder, FunctionTransformer
from sklearn.pipeline import make_pipeline
from sklearn.compose import ColumnTransformer

crosser = PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)
X_crossed = crosser.fit_transform(X_num)     # n columns become n + n(n - 1) / 2
crosser.get_feature_names_out()              # ['x0', 'x1', 'x2', 'x0 x1', 'x0 x2', 'x1 x2']
```

The categorical case has no single transformer behind it. Nothing in the library takes two category columns and returns their joint category, so the cross is built by concatenating the columns into one and encoding the result, which is the Cartesian product written out as strings:

```python
def cross_columns(df):
    joined = df.iloc[:, 0].astype(str).str.cat(df.iloc[:, 1].astype(str), sep="_x_")
    return joined.to_frame("cross")

cat_cross = make_pipeline(
    FunctionTransformer(
        cross_columns,
        feature_names_out=lambda tf, names: np.asarray(["_x_".join(names)]),
    ),
    OneHotEncoder(handle_unknown="ignore", min_frequency=20),
)

preprocessing = ColumnTransformer([
    ("cross", cat_cross, ["ocean_proximity", "income_cat"]),
    ("num", crosser, ["median_income", "housing_median_age"]),
])
```

Keeping it inside the [[Pipeline]] is what makes the vocabulary a fitted parameter learned on the training folds alone, and `min_frequency` is where the support-per-level argument above is cashed out, since it folds the levels that hold too few rows into one column instead of giving each its own coefficient. A cross that needs state of its own, ex. a list of the crossed levels worth keeping, decided from counts measured at fit time, is a [[Custom Transformer]] rather than a wrapped function.
