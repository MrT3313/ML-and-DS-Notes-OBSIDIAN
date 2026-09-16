---
note_kind: concept
aliases:
  - moment
  - moments
  - moment of a distribution
  - nth moment
  - n-th moment
  - higher moments
  - raw moment
  - raw moments
  - moment about the origin
  - central moment
  - central moments
  - moment about the mean
  - standardized moment
  - standardized moments
  - standardised moment
  - standardized central moment
  - third standardized moment
  - fourth standardized moment
  - absolute moment
  - kurtosis
  - excess kurtosis
  - leptokurtic
  - platykurtic
up: "[[Mathematics]]"
sources:
confidence: draft
---

## Definition

A moment is the expected value of a power of a random variable: one number summarizing the shape of a distribution at a chosen order, where order one fixes where it sits, order two how wide it is, order three how lopsided, order four how much weight is out in the tails. The same quantity comes in three versions differing only in where you measure from and whether you divide the units out: raw moments measure from the origin, central moments from the mean, standardized moments from the mean in units of standard deviations.

## Formal statement

For a random variable $X$ with mean $\mu = \mathbb{E}[X]$ and standard deviation $\sigma$, the $n$-th raw, central and standardized moments are

$$\mu'_n = \mathbb{E}\big[X^{n}\big], \qquad \mu_n = \mathbb{E}\big[(X - \mu)^{n}\big], \qquad \tilde{\mu}_n = \mathbb{E}\!\left[\left(\frac{X - \mu}{\sigma}\right)^{\! n}\right] = \frac{\mu_n}{\sigma^{n}}$$

Each is built from the one before it. Expanding $(X - \mu)^n$ by the binomial theorem writes any central moment as a polynomial in the raw moments,

$$\mu_n = \sum_{k=0}^{n} \binom{n}{k} (-1)^{n-k} \mu^{\,n-k} \mu'_k$$

which at $n = 2$ is the familiar $\mu_2 = \mu'_2 - \mu^{2}$. The standardized moment is then just the central one divided by $\sigma^{n}$, and $\sigma = \sqrt{\mu_2}$ is itself read off the second central moment, so nothing new enters after the raw moments and the mean.

### The low-order moments

| $n$ | raw $\mu'_n$ | central $\mu_n$ | standardized $\tilde{\mu}_n$ |
|-----|---------------------------|--------------------------------|-------------------------------------|
| $0$ | $1$ | $1$ | $1$ |
| $1$ | $\mu$, the mean | $0$ | $0$ |
| $2$ | $\mu^{2} + \sigma^{2}$ | $\sigma^{2}$, the variance | $1$ |
| $3$ | | $\mathbb{E}\big[(X-\mu)^{3}\big]$ | $\gamma_1$, skewness |
| $4$ | | $\mathbb{E}\big[(X-\mu)^{4}\big]$ | $\beta_2$, kurtosis |

The zeroth moment is $\mathbb{E}[X^0] = \mathbb{E}[1] = 1$ for every distribution, in all three versions, so it distinguishes nothing and carries no information. The first central moment is $0$ and the second standardized moment is $1$ for the same structural reason: subtracting $\mu$ and dividing by $\sigma$ is exactly what makes them so. The first informative moment therefore arrives at a different order in each column, raw at order $1$, central at order $2$, standardized at order $3$. That is the whole reason skewness is the *third* standardized moment and kurtosis the fourth: the lower standardized orders have already been spent pinning down location and scale, and only from three onward is anything left to say about shape.

Kurtosis is conventionally reported in **excess** form, $\gamma_2 = \beta_2 - 3$, because the normal distribution has $\beta_2 = 3$ whatever its $\mu$ and $\sigma$, so subtracting $3$ puts the normal at $0$ and makes the sign readable: positive is heavier-tailed than normal (leptokurtic), negative lighter-tailed (platykurtic). The fourth power weights far-out values most heavily, so kurtosis is driven by the tails; the older gloss of it as peakedness is unreliable and worth distrusting. `pandas.DataFrame.kurt()` returns the excess, bias-adjusted form, matching the convention it uses for skew.

### Sample versions

Given $m$ instances with values $x^{(i)}$ and sample mean $\bar{x} = \frac{1}{m}\sum_i x^{(i)}$, the plug-in estimators are

$$\hat{\mu}'_n = \frac{1}{m}\sum_{i=1}^{m} \big(x^{(i)}\big)^{n}, \qquad \hat{\mu}_n = \frac{1}{m}\sum_{i=1}^{m} \big(x^{(i)} - \bar{x}\big)^{n}, \qquad \hat{\tilde{\mu}}_n = \frac{\hat{\mu}_n}{\hat{\mu}_2^{\,n/2}}$$

The difference that matters is bias. $\hat{\mu}'_n$ is unbiased, but every central estimator from $n = 2$ up is not, because $\bar{x}$ was computed from the same data and sits slightly closer to the values than $\mu$ does. For $n = 2$, $\mathbb{E}[\hat{\mu}_2] = \frac{m-1}{m}\sigma^{2}$, which is where the $1/(m-1)$ in the unbiased sample variance comes from. The standardized estimators inherit the bias, and libraries correct it with a factor depending on $m$ rather than by changing the divisor.

### Units, and why standardizing matters

$\mu'_n$ and $\mu_n$ carry the units of $X$ raised to the $n$-th power. A variance of house prices is in dollars squared, its third central moment in dollars cubed, and a third central moment of room counts is in rooms cubed. Those numbers cannot be compared to each other or read for size against any fixed threshold. Dividing by $\sigma^{n}$ cancels the units exactly, so $\tilde{\mu}_n$ is a pure number, and a skew of $1.2$ means the same shape whether the column is income or rooms per household.

The same cancellation makes standardized moments invariant under an affine change of variable: for $a > 0$ and any $b$, replacing $X$ by $aX + b$ leaves every $\tilde{\mu}_n$ unchanged, since the shift cancels in $X - \mu$ and the scale cancels between $\mu_n$ and $\sigma^{n}$. With $a < 0$ the even orders are still unchanged and the odd ones flip sign.

### Absolute moments and existence

Replacing the power with $\mathbb{E}\big[|X|^{n}\big]$ gives the $n$-th absolute moment, used when the signs of odd powers would cancel information away; $\big(\mathbb{E}|X|^{n}\big)^{1/n}$ is the $L^n$ norm of the random variable, the distributional counterpart of the [[Lp Norm]] of a fixed vector.

Moments need not exist. A moment is defined only when its integral converges absolutely, and heavy tails can push $\mathbb{E}[|X|^n]$ to infinity: Student's $t$ with $\nu$ degrees of freedom has finite moments only of order $n < \nu$, so $t$ with two degrees of freedom has no finite variance and the Cauchy distribution has no mean at all. Existence runs downward, in that $\mathbb{E}[|X|^{n}] < \infty$ forces every lower order to be finite too, so a distribution has moments up to some order and none above it. The practical trap is that a finite sample always returns a finite number for any $\hat{\tilde{\mu}}_n$, whether or not the population moment it is estimating exists.

Even when all the moments are finite they do not always pin down the distribution; the lognormal is the standard counterexample, a family of visibly different densities sharing every moment. A sufficient condition for uniqueness is that the moment generating function $M(t) = \mathbb{E}[e^{tX}]$ be finite for all $t$ in some open interval around $0$, which bounds how fast the moments may grow and is where moments and the moment generating function meet.

## Where it is used

The third standardized moment is skewness, so [[Skewed Data]] is this note cashed out at a single order: its reading of $\gamma_1 > 0$ as a long right tail, its bias-adjusted sample form and its warnings about the mean-median heuristic all rest on the definitions above, and the phrase "third standardized central moment" is just the row of the table it occupies. Because standardized moments are dimensionless, one shape number is comparable to another across columns of different units, which is what makes skew and kurtosis worth scanning column by column during [[Exploratory Data Analysis]] rather than only within a column. Affine invariance settles what preprocessing can and cannot fix: [[Feature Scaling]] is an affine map and therefore cannot move any standardized moment, and [[Standardization]] is the particular affine map that drives a column's own first central moment to $0$ and second to $1$. Standardizing a column and standardizing a moment divide by the same $\sigma$ but act on different objects, one rewriting every value in the data, the other making a single summary statistic unit-free, so the shared name is a shared mechanism rather than a shared operation. The regression metrics are second and first moments of the residuals: [[Root Mean Squared Error]] is the square root of their second raw moment, and [[Mean Absolute Error]] is their first absolute moment, the same choice of power that separates the two norms it borrows from. Taking two variables at once gives joint moments, of which the covariance behind [[Correlation]] is the mixed central moment $\mathbb{E}\big[(X - \mu_X)(Y - \mu_Y)\big]$, with Pearson's $r$ its standardized form, dimensionless and bounded for exactly the reason given above.
