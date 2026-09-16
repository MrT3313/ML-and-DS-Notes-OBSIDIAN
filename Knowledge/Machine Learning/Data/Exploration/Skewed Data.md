---
note_kind: concept
aliases:
  - skew
  - skewness
  - heavy tail
  - heavy-tailed
  - right-skewed
  - left-skewed
  - positively skewed
  - negatively skewed
  - skewed distribution
  - long tail
up: "[[Feature]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A column is skewed when its values pile up on one side of the range and thin out slowly on the other, so the distribution is asymmetric and one tail is much longer than its mirror. The tail names the skew: a long tail stretching to the right is right-skewed, and it is the common case for anything bounded below by zero, such as income, population counts, or house prices.

## Formal statement

The moment coefficient of skewness is the [[Moment|third standardized central moment]],

$$\gamma_1 = \mathbb{E}\!\left[\left(\frac{X - \mu}{\sigma}\right)^{3}\right] = \frac{\mathbb{E}\big[(X-\mu)^3\big]}{\sigma^{3}}$$

Cubing preserves sign, so values far out in the right tail contribute large positive terms: $\gamma_1 > 0$ means right-skewed, $\gamma_1 < 0$ left-skewed, $\gamma_1 = 0$ symmetric in this one respect. The sample version divides the third central moment by the cubed sample standard deviation,

$$g_1 = \frac{\frac{1}{m}\sum_{i=1}^{m}\big(x^{(i)} - \bar{x}\big)^{3}}{\left[\frac{1}{m}\sum_{i=1}^{m}\big(x^{(i)} - \bar{x}\big)^{2}\right]^{3/2}}$$

and `pandas.DataFrame.skew()` returns the bias-adjusted Fisher-Pearson form, $G_1 = \frac{\sqrt{m(m-1)}}{m-2}\, g_1$.

The usual rule of thumb, $\text{Mean} > \text{Median} > \text{Mode}$ under right skew and the reverse under left skew, is a heuristic and not a theorem. It holds for many unimodal continuous distributions and fails often enough to be worth distrusting: it breaks on multimodal distributions, on distributions with one long tail and one heavy tail, and most often on discrete ones (von Hippel, *Journal of Statistics Education* 13(2), 2005). What survives as a reading rule is weaker and more useful: the mean is dragged toward the long tail, so mean well above median in a histogram is evidence of right skew.

## Where it is used

Skew is something you find during [[Exploratory Data Analysis]], usually by reading histograms, and fix later with [[Feature Distribution Transformation]]. It is not fixed by [[Feature Scaling]]: an affine map moves and stretches a distribution without changing its shape, so $\gamma_1$ is invariant under it. A skewed column makes the mean a poor summary, which is why [[Missing Value Imputation]] uses the median by default. A skewed *target* makes [[Root Mean Squared Error]] misleading, since squaring lets the few tail instances dominate the score, and [[Mean Absolute Error]] is the more honest reading in that case. Skew is also what makes a [[Stratified Sampling|stratified]] split worth the trouble: a rare tail may be absent from a small random [[Testing Set]] entirely.

One word covers two different problems. This note is the asymmetric distribution of a numeric [[Feature]] column, a property of $\mathbf{X}$ measured by the third standardized moment $\gamma_1$ above and corrected by transforming the column; a classification target whose classes appear in very unequal numbers is [[Class Imbalance]], a property of $\mathbf{y}$ measured by the class priors and corrected by stratifying, reweighting, or scoring with something other than accuracy. Neither remedy touches the other problem.
