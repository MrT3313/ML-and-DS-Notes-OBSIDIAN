---
note_kind: method
aliases:
  - slice-based evaluation
  - slice based evaluation
  - slicing
  - data slice
  - data slices
  - slice finder
  - subgroup evaluation
  - per-slice evaluation
  - worst-group accuracy
  - Simpson's paradox
  - Simpsons paradox
  - Simpson paradox
up: "[[Performance Measure]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Slice-based evaluation cuts the evaluation set into named subsets and reports the [[Performance Measure]] on each one, instead of reporting only the number computed over all of it. A slice is any subset you can describe: rows from one country, one device type, one age band, one label, one join of two of those. The protocol is worth running whenever a single score has been accepted, because accepting a score is accepting an average, and an average is a claim about nobody in particular.

**The reason it is not optional is that a pooled score hides a small slice's score by construction, and the arithmetic says exactly how much.** For any measure that is a mean of a per-instance quantity, with slices $g$ that partition the evaluation set into sizes $n_g$ summing to $n$,

$$\hat{P} = \sum_{g} \frac{n_g}{n} \, \hat{P}_g$$

so the worst slice can be arbitrarily bad and still move the pooled number by at most its share $n_g / n$. A slice holding one percent of the rows can score zero while the pooled score falls by one point, which is inside the noise of most reported comparisons. Nothing about the pooled number is wrong; it is answering a question nobody asked, namely how the model does on a randomly drawn row. [[Class Imbalance]] is the familiar special case of the same arithmetic, the one where the slicing variable is the label itself and the small slice is a rare class, and its majority-class floor $\max_k \pi_k$ is this identity with the slice score set to zero on the class the model ignores.

The quantity that does not hide it is the minimum over slices,

$$\hat{P}_{\min} = \min_{g} \hat{P}_{g}$$

the worst-group score, which is insensitive to $n_g$ and is therefore the number to report beside the pooled one. Sagawa, Koh, Hashimoto and Liang (ICLR 2020) take the same quantity into training rather than leaving it in the evaluation, minimizing the worst-group risk directly,

$$\min_{\boldsymbol\theta} \; \max_{g} \; \mathbb{E}_{(\mathbf{x}, y) \sim p_g}\big[\ell(h_{\boldsymbol\theta}(\mathbf{x}), y)\big]$$

which is group distributionally robust optimization, and their finding worth carrying here is that strong [[Regularization]] is what makes worst-group performance reachable in the overparameterized regime even where average performance does not need it. So improving $\hat{P}_{\min}$ is a different optimization problem from improving $\hat{P}$, not a matter of trying harder at the same one.

**Which measures the identity holds for is not a technicality.** It holds exactly when the measure is an average over instances: [[Accuracy]], mean squared error, mean absolute error, log loss. It fails for [[Root Mean Squared Error]], because of the square root, and it fails for [[Precision]], [[Recall]], the [[F1 Score]] and the area under a [[ROC Curve]], whose denominators are conditioned on a class or which are not means at all. For those the pooled value is not any weighted average of the per-slice values, and the argument that a slice is hidden has to be made on the cells of the [[Confusion Matrix]] instead of on the ratio.

A slice is not a distribution shift, and the two are easy to run together because both come up as "the model does badly on data that looks like this". [[Data Mismatch]] is the distribution moving: training draws from $p_{\text{train}}$, production from $p_{\text{prod}}$, and the two differ, so the whole evaluation is estimating the wrong quantity. A slice is a subset of one distribution, conditioning on $\mathbf{x} \in S$ inside a single $p$, with every row of it a row you already hold. The link between them is that a shift can often be written as a reweighting of slices, $p_{\text{prod}}(\cdot) = \sum_g w'_g \, p(\cdot \mid g)$ with $w' \ne w$, which is why per-slice scores survive a shift that the pooled score does not: the $\hat{P}_g$ stay where they were and only the weights move.

## Algorithm

1. **Fix the predictions before slicing anything.** Out-of-fold predictions over the [[Training Set]] from `cross_val_predict` ([[Cross-Validation]]), so that no row is scored by a model that was fitted on it, or a held-out split scored once. Slicing predictions the model made about rows it was fitted on measures memorisation per slice, which is worse than useless because it is smallest exactly where the model has overfitted hardest.
2. **Choose the slicing variables.** The three routes are set out below. They are not exclusive and the usual practice is all three.
3. **Materialize each slice as a boolean mask** over the evaluation rows. Slices may overlap and need not cover: the set of rows from Germany and the set of rows with a Bachelor's degree intersect, and both are legitimate slices, which is why this is not a partition and why the pooling identity above applies only to a partition.
4. **Impose a minimum support before a slice is reported at all.** A score on seven rows is not a score. For a rate $\hat{p}$ on a slice of size $n_g$ the standard error is $\sqrt{\hat{p}(1 - \hat{p})/n_g}$, so a slice of $25$ rows at $\hat{p} = 0.5$ carries a standard error of $0.10$, and a slice that appears $15$ points worse than the pooled score is inside one of them.
5. **Compute the measure per slice and keep three columns: the score, the size, and the share.** Without the size beside it a per-slice score cannot be read, since the two ways a slice score departs from the pooled one, a real weakness and a small denominator, produce the same-looking number.
6. **Read the worst slice and the spread, not the mean of the slice scores.** The unweighted mean over slices is a different quantity from the pooled score and, with overlapping or partial slices, is not a score on any dataset at all.
7. **Test whether the gap is real before acting on it**, with a two-sample test on the per-instance losses and an effect size, and correct for multiplicity in proportion to how many slices you looked at. Searching many slices and reporting the worst is [[Data Snooping Bias]] when the search runs on the [[Testing Set]], and is a multiple-comparisons problem even when it does not.
8. **Act, then re-run the whole protocol.** Every remedy redistributes performance across slices, and a fix for the worst slice that quietly costs the pooled score is a trade to be made deliberately rather than discovered later.

### Simpson's paradox, as the inequality it is

The paradox is an arithmetic fact about weighted averages and not a curiosity about data. Compare two things, the $a$ side and the $b$ side, across two slices. The $a$ side gets $a_1$ of $n_1$ right in slice 1 and $a_2$ of $n_2$ in slice 2; the $b$ side gets $b_1$ of $m_1$ and $b_2$ of $m_2$. All three of these can hold at once:

$$\frac{a_1}{n_1} > \frac{b_1}{m_1}, \qquad \frac{a_2}{n_2} > \frac{b_2}{m_2}, \qquad \text{and yet} \qquad \frac{a_1 + a_2}{n_1 + n_2} < \frac{b_1 + b_2}{m_1 + m_2}$$

The $a$ side wins both slices and loses the pool. Why that is possible is visible the moment each pooled ratio is rewritten as what it is, a size-weighted average of its own two slice ratios. Put $p_g = a_g/n_g$, $q_g = b_g/m_g$, and let

$$w = \frac{n_1}{n_1 + n_2}, \qquad v = \frac{m_1}{m_1 + m_2}$$

so that

$$\frac{a_1 + a_2}{n_1 + n_2} = w\,p_1 + (1 - w)\,p_2, \qquad \frac{b_1 + b_2}{m_1 + m_2} = v\,q_1 + (1 - v)\,q_2$$

**The two pooled numbers are averages of the same two comparisons taken under different weights, and that is the entire mechanism.** If $w = v$ the reversal is impossible: the pooled comparison is then a convex combination of $p_1 > q_1$ and $p_2 > q_2$ with the same coefficients on both sides, so it inherits their direction term by term. Unequal slice sizes across the two sides are therefore a necessary condition, and the reversal happens when the side that wins the pool is the one putting its weight on the easier slice.

That necessary condition is sharper than the usual telling, and it settles what the paradox can and cannot do to a model comparison. Two candidate models scored on the *same* evaluation set have identical slice sizes, so for any measure whose denominator is the slice size, [[Accuracy]] and mean squared error among them, $w = v$ and no pooled comparison of the two models can reverse their per-slice comparisons. That qualification is load bearing: [[Precision]]'s denominator is the model's own count of predicted positives rather than the size of the slice, so two models scored on one evaluation set genuinely do carry different weights there and the reversal is available again. The other two cases are the ones where the weights differ outright, comparing two *groups of rows* inside one model's predictions, where the groups really are composed differently, and comparing two models scored on differently composed sets, which is a defect in the comparison before the paradox is reached.

The standing instance of the group case is the Berkeley graduate admissions data of autumn 1973, reported by Bickel, Hammel and O'Connell (*Science* 187, 1975). Campus-wide, $8442$ men applied and about $44$ percent were admitted, against $4321$ women and about $35$ percent, which reads as an admissions process that favours men. Take the two departments at the extremes of the six largest, one admitting most of its applicants and one admitting under a tenth of them, and the reversal is exact. The women's counts are the $a$'s and the men's the $b$'s:

$$\frac{89}{108} = 0.824 > \frac{512}{825} = 0.621, \qquad \frac{24}{341} = 0.070 > \frac{22}{373} = 0.059$$

$$\text{and yet} \qquad \frac{89 + 24}{108 + 341} = 0.252 \;<\; \frac{512 + 22}{825 + 373} = 0.446$$

Women were admitted at the higher rate in each department separately and at barely half the men's rate over the two of them together. The weights are the whole of it: only $w = 108/449 = 0.24$ of the women's applications went to the easier department, the one admitting $62$ percent of the men who applied to it and $82$ percent of the women, against $v = 825/1198 = 0.69$ of the men's. Over all six of the largest departments, four admitted women at the higher rate while the pooled figures were $1198$ of $2691$ men, $44.5$ percent, against $557$ of $1835$ women, $30.4$ percent. The variable that produced the reversal, which department was applied to, is a variable the pooled comparison had marginalized out.

Read as a model evaluation, the departments are the slices and the two sides are two subgroups whose slice mixes differ. The lesson is not that pooling is wrong, it is that a pooled comparison between two differently composed groups answers a question about the composition as much as about the model, and only the per-slice table separates the two.

### Where the slices come from

Slicing is more art than science, and the three routes in ordinary use differ in how much of the work is yours.

- **Heuristics.** Domain knowledge names the variable: the app's two platforms, paying and non-paying accounts, the languages the product is sold in, the label itself. This is the cheapest route and it finds the slices somebody will ask about in a review, which is a different set from the slices where the model is actually weakest.
- **[[Error Analysis]].** Read the misclassified instances and name what they have in common; the pattern you name is a slice, and going back to measure the model on it is this protocol. This is the same operation the automated tools perform, done by hand, and it is the only route that can name a slice no column in the table expresses.
- **Automated slice finding.** Search the space of candidate slices for the ones that are both problematic and large. Slice Finder (Chung, Kraska, Polyzotis, Tae and Whang, ICDE 2019) defines a slice as a conjunction of feature-value predicates, which is what keeps it interpretable and therefore actionable, and screens each candidate $S$ against its complement $S'$ on two tests at once. Statistical significance comes from Welch's $t$ statistic on the per-instance losses, and magnitude from an effect size,

  $$t = \frac{\mu_S - \mu_{S'}}{\sqrt{\sigma_S^2/|S| + \sigma_{S'}^2/|S'|}}, \qquad \phi = \sqrt{2} \cdot \frac{\psi(S, h) - \psi(S', h)}{\sqrt{\sigma_S^2 + \sigma_{S'}^2}}$$

  with $\psi$ the average loss on a slice and $\sigma^2$ the variance of the individual instance losses in it. Both tests are needed, because significance says an effect exists and $\phi$ says whether it is large enough to spend a week on: $\phi = 1$ means the two loss distributions sit one standard deviation apart, and Cohen's thresholds put $0.2$ at small, $0.5$ at medium, $0.8$ at large. Because the search examines a great many slices, it controls the marginal false discovery rate by $\alpha$-investing rather than reporting everything that passes. In this vault $\phi$ is a Slice Finder effect size here and nothing else; the feature map of [[Feature Engineering]] and the Shapley value of [[Feature Importance]] are unrelated uses of the same letter.

The route matters less than the fact that all three end in the same place, a table of slices with a score and a size each, read by the protocol above.

## Hyperparameters

None. The thresholds that decide which slices come back, the minimum support, the effect size threshold $T$ and the significance level, belong to whichever slice finder is doing the searching rather than to the protocol of reading the table, and they are stated with the finder above.

## Failure modes

- **Slicing so finely that every slice is noise.** With small $n_g$ the standard error $\sqrt{\hat{p}(1 - \hat{p})/n_g}$ swamps the gap being argued about, and since the number reported is the *minimum* over slices, finer slicing makes the picture look worse whether or not anything is wrong. The minimum support in step 4 is not a nicety, it is what keeps the reading from being an artefact of the cut.
- **Reporting the worst of many searched slices without correcting for multiplicity.** The minimum of many noisy estimates is biased low by construction: search enough slices of a model with no real weakness and the worst one still looks bad. This is why Slice Finder carries $\alpha$-investing, and it becomes [[Data Snooping Bias]] outright when the search runs on the [[Testing Set]] and a modelling decision follows.
- **Slicing on a variable computed from the model's own output.** Grouping by predicted class and reading the per-group accuracy is a column of the [[Confusion Matrix]], not a subgroup score, and conditioning on what the model said cannot support a claim about a population. The slicing variable has to be something the row carried before the model saw it.
- **Taking the unweighted mean over slices as the model's score.** It is not the pooled score, it is not a score on any dataset when the slices overlap, and it silently reweights the evaluation set towards whichever slices happen to be small.
- **Fixing the worst slice and not re-reading the pooled number.** Worst-group and average performance are different objectives, as the group distributionally robust result makes explicit, so the trade is real and has to be shown as two numbers rather than reported as an improvement.
- **Slicing predictions the model was fitted on.** The per-slice scores are then smallest exactly where the model memorized hardest, so the protocol points at the wrong slices, which is the same defect [[Error Analysis]] guards against with out-of-fold predictions.

## Implementation

**There is no slice-finding API in scikit-learn 1.6, and no per-slice evaluation API either.** The honest statement of what the library gives you is: `cross_val_predict` for held-out predictions, a `groupby` over the slicing columns, and any function from `sklearn.metrics` computed inside the groups. That is the whole implementation, and it is short enough that the absence of a dedicated call costs nothing.

scikit-learn 1.6 and pandas 2.x, on [[California Housing]]:

```python
import numpy as np
import pandas as pd
from sklearn.datasets import fetch_california_housing
from sklearn.ensemble import HistGradientBoostingRegressor
from sklearn.model_selection import cross_val_predict, train_test_split

X, y = fetch_california_housing(as_frame=True, return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# step 1: every row predicted by a model fitted on folds that excluded it
y_oof = cross_val_predict(HistGradientBoostingRegressor(random_state=42),
                          X_train, y_train, cv=5)

# steps 2 and 3: two slicing variables, one banded and one a domain heuristic
frame = pd.DataFrame({
    "squared_error": (y_oof - y_train.to_numpy()) ** 2,
    "age_band": pd.cut(X_train["HouseAge"], [0, 15, 30, 45, 55]).to_numpy(),
    "inland": (X_train["Longitude"] < -120.5).to_numpy(),
})

# steps 4 and 5: score, size and share, with a support floor
per_slice = frame.groupby("age_band", observed=True)["squared_error"].agg(["mean", "size"])
per_slice["share"] = per_slice["size"] / len(frame)
pooled = frame["squared_error"].mean()

# the pooling identity, exactly: size-weighted slice means reproduce the pooled mean
assert np.isclose((per_slice["mean"] * per_slice["size"]).sum() / per_slice["size"].sum(),
                  pooled)

# step 6: the worst slice and the spread, not the mean of the slice scores
print(per_slice.sort_values("mean", ascending=False))
print("pooled MSE", round(pooled, 4), "worst slice MSE", round(per_slice["mean"].max(), 4))

# two variables at once, which is where a slice finder would look, with the floor applied
crossed = (frame.groupby(["age_band", "inland"], observed=True)["squared_error"]
                .agg(["mean", "size"]))
print(crossed[crossed["size"] >= 100].sort_values("mean", ascending=False))
```

The `assert` is the pooling identity from the top of this note, and it passes for mean squared error and fails if `mean` is swapped for a root mean squared error, which is the cheapest demonstration of why the choice of measure decides whether the weighted-average argument is available at all.

The nearest thing to a packaged version is **fairlearn** (0.12 or later), whose `MetricFrame` is exactly a disaggregated metric: it takes `y_true`, `y_pred` and `sensitive_features`, and exposes `overall`, `by_group`, `group_min()`, `group_max()`, `difference()` and `ratio()`, which is steps 5 and 6 as an object. Its `control_features` argument is the crossed case above, stratifying rather than aggregating over the second variable.

```python
from fairlearn.metrics import MetricFrame
from sklearn.metrics import mean_squared_error

mf = MetricFrame(metrics=mean_squared_error, y_true=y_train, y_pred=y_oof,
                 sensitive_features=frame["age_band"])
mf.overall, mf.by_group, mf.group_max()
```

The tools that search for slices rather than scoring the ones you name are separate, and all three are research artefacts rather than library calls. **Slice Finder** (Chung, Kraska, Polyzotis, Tae and Whang, ICDE 2019) is the prototype whose statistics are given above; the formulas come from the authors' extended version rather than the four-page conference paper. **SliceLine** (Sagadeeva and Boehm, *SIGMOD* 2021) recasts the same search as sparse linear algebra over a recoded and binned feature matrix and an error vector, returning the top $k$ slices under a score that trades a slice's average error against its size with a weight $\alpha$; it ships as a builtin function in Apache SystemDS. **Robustness Gym** (Goel, Rajani, Vig, Taschdjian, Bansal and Ré, NAACL 2021 demonstrations) treats subpopulations as one of four evaluation paradigms behind a single interface, the other three being transformations, evaluation sets and adversarial attacks, which puts slices and [[Behavioral Testing]] in one harness.
