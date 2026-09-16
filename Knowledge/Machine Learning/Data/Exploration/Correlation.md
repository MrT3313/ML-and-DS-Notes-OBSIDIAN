---
note_kind: concept
aliases:
  - Pearson's r
  - Pearson correlation
  - correlation coefficient
  - standard correlation coefficient
  - linear correlation
  - correlations
  - correlation matrix
up: "[[Feature]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

The standard correlation coefficient, Pearson's $r$, is one number between $-1$ and $1$ saying how closely two numeric columns track each other along a **straight line**. Near $+1$ one rises as the other rises, near $-1$ one rises as the other falls, near $0$ there is no *linear* relationship, which is a far weaker statement than "no relationship".

## Formal statement

For paired observations $(x_i, y_i)$ with means $\bar{x}$ and $\bar{y}$,

$$r = \frac{\sum_{i}(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum_{i}(x_i - \bar{x})^2}\sqrt{\sum_{i}(y_i - \bar{y})^2}}$$

with $r \in [-1, 1]$. It is the covariance of $x$ and $y$ divided by the product of their standard deviations, covariance rescaled into a unit-free number, unchanged by shifting or positively rescaling either variable.

Two consequences matter more than the formula. First, $r$ sees lines and nothing else. A strong nonlinear relation, a parabola, a U shape, a threshold where $y$ jumps once $x$ passes some value, gives $r \approx 0$ because the rising half cancels the falling half. Dropping a [[Feature]] on a near-zero $r$ therefore discards signal a nonlinear model would have used. Near-zero $r$ is a reason to plot the pair, not to delete the column.

Second, $r$ says nothing about slope or about the shape of the point cloud. A gentle trend and a steep one both reach $r = 1$ if the points sit on a line, and wildly different scatter plots share identical $r$. [Anscombe's quartet](https://en.wikipedia.org/wiki/Anscombe%27s_quartet) (four datasets with matching $r$, means, and variances but four unrelated shapes) and its successor the [Datasaurus Dozen](https://en.wikipedia.org/wiki/Datasaurus_dozen) are the standard demonstrations. Correlation is also not causation: two columns can move together because a third drives both.

Spearman rank correlation applies the same formula to the ranks instead of the values, so it reaches $\pm 1$ for any strictly monotonic relation, curved or not.

```python
corr_matrix = housing.corr(numeric_only=True)
corr_matrix["median_house_value"].sort_values(ascending=False)

housing.corr(method="spearman", numeric_only=True)   # monotonic instead of linear
```

## Where it is used

In [[Exploratory Data Analysis]] the correlation matrix is the first bivariate view after the histograms, and a scatter matrix is how you check whether a given $r$ tells the truth about the shape. It is the standard screen for [[Irrelevant Features]], subject to the linearity caveat above. Its payoff in the housing chapter is [[Feature Engineering]]: `median_income` correlates with the target at about $0.69$ while `total_bedrooms` manages roughly $0.05$, yet the derived ratio of bedrooms to rooms reaches about $-0.26$, so a feature combining two weak columns beats either parent. And because [[Linear Regression]] fits exactly the straight-line structure $r$ measures, $r$ is a fair advance estimate of what one feature buys that model, and an unfair one for any model that can bend.
