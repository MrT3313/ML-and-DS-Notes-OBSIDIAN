---
note_kind: concept
aliases:
  - irrelevant feature
  - garbage in garbage out
  - garbage in, garbage out
  - GIGO
up: "[[Feature]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Irrelevant features are input columns that carry no information about the target. They slow training, give the model noise to fit, and worsen [[Generalization]]. The remedy is feature engineering: keep the useful features, combine existing ones into more useful ones, and gather new ones.

## Formal statement

A [[Feature]] $x_j$ is irrelevant when $y \perp x_j \mid \mathbf{x}_{-j}$, so no model gains from it. Feature engineering is a map $\phi: \mathbb{R}^{n} \to \mathbb{R}^{n'}$ chosen so that $y$ is easier to predict from $\phi(\mathbf{x})$.

That statement is about the population, and no finite sample settles it directly. What a sample does support is a measurement of what a fitted model loses when the column goes. Write $\mathcal{L}$ for risk estimated on held-out rows, $h$ for the model fitted with every column present, and $h_{-j}$ for the same model with column $j$ removed or destroyed:

$$I_j \;=\; \mathcal{L}\big(h_{-j}\big) - \mathcal{L}(h)$$

Conditional independence forces $I_j = 0$ in the population, so a column whose measured $I_j$ sits inside the spread of the estimate is the sample-level reading of irrelevance, and thresholding on it is the selection criterion. The reading runs one way only. A large $I_j$ is evidence the column carries something; a small $I_j$ is consistent with genuine irrelevance and equally consistent with a column that some other column duplicates, since $\mathbf{x}_{-j}$ still holds what was taken away and the model simply reads it there instead. Two copies of one useful column therefore both score near zero and dropping either alone costs nothing, which is why a per-column sweep is a screen and not a proof. [[Feature Importance]] carries the two ways of building $h_{-j}$, refitting without the column against permuting it in held-out rows, and the difference between what they measure.

A second criterion reads the column rather than the fit. The empirical mutual information between column and target,

$$\hat{I}(x_j; y) \;=\; \sum_{u}\sum_{v} \hat{p}(u, v) \log \frac{\hat{p}(u, v)}{\hat{p}(u)\,\hat{p}(v)}$$

is cheap and model-free, and it answers a weaker question than the definition asks, because it is marginal: it is zero when $y \perp x_j$ outright, not when $y \perp x_j \mid \mathbf{x}_{-j}$, so it keeps a column that is informative alone and redundant in company. A third route makes the criterion part of the fit rather than a step before it, by adding an $\ell_1$ penalty $\alpha \lVert \boldsymbol\theta \rVert_1$ to the [[Cost Function]] and reading off the coordinates driven exactly to zero, which selects conditionally, in the presence of the other columns, at the price of being tied to one model family.

A column can also fail on availability rather than on information, being informative on the rows that have it and absent from most of them. That is coverage, and it belongs to [[Feature Generalization]].

## Where it is used

[[Feature Engineering]] holds the account of what can be done to a feature table and how those operations divide. The branch this pathology calls for is selection, dropping the columns that do not pay, with [[Dimensionality Reduction]] as the neighbouring answer that replaces the columns rather than discarding them. Interacts with [[Overfitting]]: more irrelevant columns give a flexible model more noise to memorize.

### What carrying a column costs

More features usually buys more performance, and past some point the count itself is the problem. The costs are not all of one kind and they are not all paid at the same moment, which is the part worth separating.

**Two are paid while training, and they are about accuracy.** Every extra column is another chance for [[Data Leakage]], since the surface on which a column computed from the answer can slip in grows with the number of columns, and a wide table is audited less carefully than a narrow one. And every extra column widens the space a flexible model has to memorize noise in, which is [[Overfitting]] and the reason the conditional independence above is worth acting on rather than merely stating.

**Two are paid at serving time, and they are about resources rather than about accuracy.** A column in the model is a column the deployed system has to compute and hold for every request, so the memory footprint of serving grows with the feature count whether or not the extra columns earn anything. And each one has to be computed before the prediction can be returned, which adds to the time a request takes; a feature assembled from a lookup or a join adds more of it than one read straight off the request. Neither cost shows anywhere in a validation score, which is why a table that is chosen on accuracy alone tends to be wider than it should be.

**One is paid afterwards, and it is about maintenance.** A feature that has stopped paying does not remove itself. It still has to be computed, monitored and kept alive, its upstream source still has to keep producing it, and nothing raises an error on the day it stops earning, so the cost carries on being paid long after the benefit has gone. That is a claim about the work of keeping a system running rather than about the accuracy of a fit, and it is the reason removing a column that no longer contributes is work worth scheduling rather than work that will happen on its own.
