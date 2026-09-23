---
note_kind: method
aliases:
  - bagging
  - bootstrap aggregating
  - bootstrap aggregation
  - bagged
  - bagged trees
  - out-of-bag
  - out of bag
  - OOB
  - oob_score
  - BaggingClassifier
  - BaggingRegressor
up: "[[Ensemble Learning]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Bagging fits one algorithm many times over, each time on a different resample of the training rows drawn with replacement, and aggregates the fitted models into a single predictor: the average of their outputs for [[Regression]], a vote for [[Classification]]. Nothing about the algorithm, the features or the hyperparameters changes between members. The only thing that differs is which rows each member happened to see, and that difference alone is enough to make the members disagree, which is the resource [[Ensemble Learning]] says every ensemble spends and this method manufactures the cheapest way available.

Reach for it when the base learner is unstable, meaning a small perturbation of the training rows produces a visibly different fitted model. That is the condition the whole method rests on, and it is a property of the learning procedure rather than of the data. A fully grown decision tree is the standing case: it partitions the rows until the leaves are pure, so moving a handful of rows changes which cut is chosen near the root and everything below it, and the resulting model has low bias and high variance ([[Overfitting]]). Bagging keeps the flexibility and cancels part of the noise. It does nothing for a learner whose fit barely moves when the rows are perturbed, and the reason is the same one stated the other way round: with nothing to average out, all that is left is the cost of $M$ fits.

Against its sibling: bagging is parallel and [[Boosting]] is sequential. Every member here is fitted independently of every other, in any order, on a resample drawn without reference to how the previous members did, so the mechanism attacks variance and leaves bias where it found it. Boosting fits members one after another, each conditioned on the errors of its predecessors, which is what lets it attack bias instead. The two are not competing settings of one dial; they are different mechanisms aimed at different terms of the error.

## Algorithm

Write the training split as $D$ with $n$ rows, the base learner as a procedure that turns a set of rows into a fitted model, and $M$ for the number of members.

1. **Fix $M$**, the number of members to fit.
2. **Draw the resample.** For member $j$, draw $n$ rows from $D$ uniformly at random *with replacement*, giving a bootstrap resample $D_j$ of the same size as $D$. Rows repeat inside $D_j$, and about a third of the rows of $D$ are absent from it altogether, which is derived below.
3. **Fit member $j$** on $D_j$ alone, with no reference to any other member and no reference to how any other member did. The loop over $j$ is therefore order-independent and parallel, and refitting one member changes nothing about the rest.
4. **Repeat steps 2 and 3** for $j = 1, \dots, M$.
5. **Aggregate.** For regression, average the members' predictions,
   $$\hat{y}(\mathbf{x}) = \frac{1}{M}\sum_{j=1}^{M} \hat{h}_j(\mathbf{x})$$
   For classification, either take the plurality vote over the members' predicted labels, or average their predicted class probabilities and take the largest, which is the route scikit-learn takes whenever the base learner exposes `predict_proba`. The two do not always agree: a vote discards how confident each member was, so a single certain member can be outvoted by several members that were barely leaning.
6. **Optionally score the ensemble out of bag**, using the rows each member did not see. This is free and is the subject of its own subsection below.

### What a resample leaves out

The one number to know about step 2 is what fraction of the distinct rows of $D$ a resample of size $n$ actually contains. Each of the $n$ draws lands on row $i$ with probability $1/n$ and misses it with probability $1 - 1/n$, and the draws are independent, so all $n$ of them miss row $i$ with probability $(1 - 1/n)^{n}$ and row $i$ appears at least once with probability

$$P\big(\text{row } i \in D_j\big) = 1 - \left(1 - \frac{1}{n}\right)^{n}$$

Every row carries the same probability, so by linearity the expected number of distinct rows in $D_j$ is $n\left[1 - (1 - 1/n)^{n}\right]$ and the expected *fraction* is the bracket alone. Writing $(1 - 1/n)^{n} = \exp\!\big(n \ln(1 - 1/n)\big)$ and expanding the logarithm as $\ln(1 - 1/n) = -1/n - 1/(2n^{2}) - \dots$ gives $n \ln(1 - 1/n) \to -1$, so

$$1 - \left(1 - \frac{1}{n}\right)^{n} \;\xrightarrow[\;n \to \infty\;]{}\; 1 - e^{-1} \approx 0.6321$$

The limit is approached from above and fast enough that the asymptotic figure is the practical one: $0.6513$ at $n = 10$, $0.6340$ at $n = 100$, $0.6323$ at $n = 1000$. So a bootstrap resample of size $n$ holds roughly $0.632n$ distinct rows, the remaining draws being repeats of rows already in it, and roughly $0.368n$ rows of $D$ are **out of bag**, absent from that resample entirely.

Breiman reaches the same number by a different route, treating the number of times row $i$ is selected in $n$ draws as approximately Poisson with $\lambda = 1$ for large $n$, so that the chance of being selected at least once is $1 - e^{-1}$. The two derivations agree because the Poisson is the limit of the binomial with $n$ trials and success probability $1/n$; the binomial form above is the exact one and the Poisson is the approximation, which is worth knowing only so that the figure is not mistaken for an empirical observation.

That single quantity is both halves of the method at once, which is what makes it the central fact rather than a piece of trivia.

- **It is why bagging works.** Two members see overlapping but genuinely different two-thirds of the data, so two unstable fits on them differ, and it is that difference the aggregation converts into a lower variance.
- **It is where the out-of-bag estimate comes from.** The complementary $36.8$ percent are rows that member $j$ provably never saw, so they are a held-out sample for that member, already paid for.

### The out-of-bag estimate

Let $B_i = \{\, j : \text{row } i \notin D_j \,\}$ be the set of members that never saw row $i$. Each membership is an independent Bernoulli event with probability $(1 - 1/n)^{n} \approx e^{-1}$, so

$$\mathbb{E}\big[|B_i|\big] \approx \frac{M}{e} \approx 0.368\,M$$

The out-of-bag prediction for row $i$ aggregates over those members only,

$$\hat{y}^{\,\text{oob}}_i = \frac{1}{|B_i|}\sum_{j \in B_i} \hat{h}_j\big(\mathbf{x}^{(i)}\big)$$

and the out-of-bag score is whatever [[Performance Measure]] is wanted, computed from those predictions against the true labels. Every prediction entering it comes from a fit that excluded the row being predicted, which is exactly the property `cross_val_predict` is used to buy in [[Cross-Validation]], obtained here as a by-product of the resampling instead of by fitting $K$ further models. That is the whole attraction: an estimate with the same honesty as a cross-validated one, at no extra fitting cost.

Two qualifications follow from the arithmetic above rather than from experience. The estimate describes an ensemble of about $0.368M$ members and not one of $M$, so at small $M$ it is pessimistic about the ensemble you actually have, and the gap closes as $M$ grows. And a row has no out-of-bag prediction at all when $|B_i| = 0$, which happens with probability $(1 - e^{-1})^{M} \approx 0.632^{M}$: about one row in a hundred at $M = 10$, about one in ten thousand at $M = 20$, and about $10^{-20}$ at $M = 100$. Small ensembles therefore produce an out-of-bag score computed over a subset of the rows, silently.

How good the estimate is was measured rather than assumed. Breiman's out-of-bag technical report benchmarks it against the best that the same number of rows could buy, an independent test set of the size of the training set, and finds the ratio of the two errors close to one across nine classification and five regression data sets, with the classification estimates close to unbiased and the regression estimates possibly systematically low. His position in the random forest paper is stronger still, that this removes the need for a set-aside test set. Where that position has to be qualified is stated as a failure mode below, and the qualification is not about the accuracy of the estimate: it is about what happens once candidates are selected by it.

### What averaging can and cannot fix

[[Ensemble Learning]] carries the variance of an average of $M$ members with per-member error variance $\sigma^{2}$ and average pairwise error correlation $\rho$, namely $\rho\sigma^{2} + \frac{1-\rho}{M}\sigma^{2}$, and bagging is read straight off it without rederiving anything. The resampling is a device for lowering $\rho$; $M$ is the dial on the second term and on nothing else.

Both readings of that formula bite here. Resampling cannot drive $\rho$ to zero, because the bags are drawn from one file and therefore overlap: two independently drawn bags share a row with probability $(1 - e^{-1})^{2} \approx 0.40$, so about forty percent of the distinct rows are common to any two members and their fits are correlated through that common material. The floor $\rho\sigma^{2}$ is where a bagged ensemble stops, and adding members past the point where $\frac{1-\rho}{M}\sigma^{2}$ is small against it buys nothing measurable.

What bagging does not touch is the bias. Every member is fitted on a resample of the same distribution and therefore carries approximately the same bias as a single fit would, and an average of $M$ equally biased predictors has that bias intact; the [[Bias-Variance Tradeoff]] decomposition is where those two terms are separated. This is the precise sense in which bagging an [[Underfitting]] learner is wasted work: the term it attacks is already small and the term that is large is untouched.

### Instability is the condition, and it cuts both ways

The argument that makes instability the condition is worth having in full, because the slogan version loses the half that says bagging can hurt. Breiman's original ("Bagging Predictors", *Machine Learning* 24, 1996, freely readable as Berkeley Statistics Technical Report 421, 1994) does not argue from resamples at all. Write $\varphi(\mathbf{x}, L)$ for the model a procedure produces from a learning set $L$, and $\varphi_A(\mathbf{x}) = \mathbb{E}_L\,\varphi(\mathbf{x}, L)$ for the average of that model over learning sets drawn from the true distribution. Under squared error, the inequality $(\mathbb{E}Z)^{2} \le \mathbb{E}Z^{2}$ applied pointwise gives

$$\mathbb{E}_L\,\mathbb{E}_{Y,\mathbf{X}}\big(Y - \varphi(\mathbf{X}, L)\big)^{2} \;\ge\; \mathbb{E}_{Y,\mathbf{X}}\big(Y - \varphi_A(\mathbf{X})\big)^{2}$$

so the aggregated predictor is never worse than the single one, and the size of the improvement is exactly the slack in that inequality, which is exactly how much $\varphi(\mathbf{x}, L)$ moves when $L$ is redrawn. A procedure whose fit barely moves leaves almost no slack to collect. This is the same content as the variance reading above, though not the same vocabulary: the words in that paper are instability and the gap between the two sides, and neither "variance" nor Jensen's name appears in it.

The catch is that bagging does not compute $\varphi_A$. It averages over resamples of the one file in hand rather than over learning sets from the distribution, so what it computes is $\varphi_A$ under the empirical distribution, and Breiman's image for the consequence is that the bagged predictor is caught between two currents: an unstable procedure gains through the aggregation, while for a stable procedure the empirical substitute is simply a worse estimate than the single fit it replaced. That is the mechanism of the degradation, and it is why the honest summary is not that bagging helps a little or a lot but that it helps or hurts. His own summary of the classification case, verbatim: "Bagging unstable classifiers usually improves them. Bagging stable classifiers is not a good idea."

The never-worse guarantee also does not survive the move to classification. It is derived under squared error, where the aggregate is an average; under voting the aggregate is not a linear function of the members, and Breiman states the asymmetry directly, that aggregating turns good predictors into nearly optimal ones while poor predictors can be turned into worse ones. A committee of members that are each wrong more often than right in some region votes unanimously for the wrong answer there.

**Which procedures fall on which side**, as that paper has it: neural networks, classification and regression trees, and subset selection in linear regression are the unstable ones, and nearest-neighbour methods are stable. Two things about that list are worth carrying, because it is usually repeated without them. Neural networks are in it on the authority of an earlier technical report of Breiman's on the heuristics of instability in model selection, not on any experiment in the bagging paper, whose experiments are classification trees, regression trees, subset selection in linear regression and nearest neighbours and nothing else. And the degradation claim is made about stable procedures in general rather than about nearest neighbours in particular: the nearest-neighbour experiment there reports bagged and unbagged error rates identical to one decimal place on all six of its data sets, so nearest neighbours is that paper's example of a procedure bagging leaves *unchanged*, while the procedure it actually measures getting worse is subset selection in linear regression when many variables are retained.

One more number from the same experiments, which is the empirical counterpart of the $1/M$ decay in the table below: most of the available improvement was already in hand at ten bootstrap replicates, and beyond twenty five there was nothing left to collect on those data sets. That is a reading from those problems and not a constant, and the estimators discussed below default to far more, but it is the right order of magnitude to expect the curve to flatten at.

### Random forests

The random forest is the standing instance of bagging and the reason the method is used as widely as it is. It is a collection of decision trees fitted by bagging, with one further source of randomness laid on top: at each split, instead of searching every feature for the best cut, the tree searches only a random subset of the features, redrawn at every node. Two trees in a forest therefore differ twice over, in the rows they were fitted on and in the features they were permitted to consider at each node, and it is the second of those that drives the correlation between their errors below what resampling alone reaches. Breiman's two stated reasons for keeping the bagging inside it are that it improves accuracy when random features are used, and that it supplies the out-of-bag machinery above, which is what the forest's error, strength and correlation estimates are computed from.

How many features to try at a node is a parameter and not a constant. Breiman's "Random Forests" (*Machine Learning* 45, 2001) calls its size $F$ and runs two settings, $F = 1$ and $F$ the first integer below $\log_2 p + 1$ for $p$ inputs, reporting the results insensitive to the choice and one or two features usually near optimal. The familiar $\sqrt{p}$ is not in that paper: it is the default of Breiman's later software and of the libraries that followed it, which is why it appears in the table below as a scikit-learn fact rather than as a recommendation from the paper.

The price of the feature restriction is stated in the same two quantities as before. Withholding features at a node makes each individual tree worse, so $\sigma^{2}$ rises while $\rho$ falls, and the forest beats a bagged ensemble of unrestricted trees exactly when the second effect outweighs the first. That paper makes the trade exact, with a strength $s$ defined as the expected margin of a random member and a mean correlation $\bar{\rho}$ between the members' raw margin functions, giving a Chebyshev bound on the generalization error of the forest:

$$PE^{*} \le \frac{\bar{\rho}\,(1 - s^{2})}{s^{2}}$$

Two readings, and Breiman's own warning that the bound is likely to be loose, so it is suggestive rather than a number to quote. The error is controlled by exactly two things, how strong the members are and how correlated they are, which is the same pair the averaging formula names. And the ratio $\bar{\rho}/s^{2}$ is the quantity to drive down, so a change that weakens the trees is worth making only if it decorrelates them faster than it weakens them.

The forest's other theoretical result is about the number of trees. As members are added the generalization error converges almost surely to a limit, by the strong law of large numbers, which is why adding trees to a forest does not overfit. The scope of that is worth being exact about: it says the error settles rather than diverging with $M$, and it says nothing about overfitting through the other controls, since growing each tree to maximum depth on a small file overfits by a route this theorem does not address.

The tree as a model in its own right, its splitting criterion, the controls on its depth and the feature importances read off a fitted forest, arrives with HOML chapters 6 and 7. What is here is the ensemble mechanism alone.

On provenance: both of Breiman's journal papers cited above are paywalled, and every claim attributed to them here was checked against his own Berkeley Statistics technical reports, Technical Report 421 of September 1994 for the bagging paper and the January 2001 report for the random forest paper, plus the undated out-of-bag report of the same period. The published abstract of the random forest paper was separately confirmed word for word against the technical report's; the bagging paper's was not independently confirmable, so treat the wording attributed to it as the technical report's wording.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of members, `n_estimators` | $M$ | `10` on `BaggingClassifier` and `BaggingRegressor`, `100` on `RandomForestClassifier` and `RandomForestRegressor` | the second term of $\rho\sigma^{2} + \frac{1-\rho}{M}\sigma^{2}$ shrinks, so the ensemble's variance falls toward the $\rho\sigma^{2}$ floor with the gain per added member decaying like $1/M$. Fit time, memory and prediction latency all grow linearly. It does not make the ensemble overfit, because members are fitted independently and never see each other's errors | raise it until the out-of-bag or cross-validated score stops improving, then stop. The curve flattens rather than turning, so the question is where the compute stops being worth it and not where the optimum is |
| resample size, `max_samples` | | `1.0`, meaning a resample of the same size as the training split | each member sees more of the data, so its own variance falls but the bags overlap more, $\rho$ rises and the ensemble's floor rises with it. Below $1.0$ the opposite trade, more diverse but individually weaker members | leave at $1.0$ unless fit time or memory forces it down, which is the usual reason to move it. Small values on a large file are a deliberate speed-for-diversity trade and are worth scoring rather than assuming |
| draw with replacement, `bootstrap` | | `True` | not a direction. `True` is bagging as derived above, with its $0.632$ coverage and its out-of-bag set. `False` draws each subset without replacement, which is pasting: no row repeats inside a member, and with `max_samples=1.0` every member gets the whole training split and the ensemble collapses to one model | keep `True` whenever the out-of-bag estimate is wanted, since without replacement and at full size there is nothing left out. Set `False` only together with a `max_samples` well below $1.0$ |
| features per member, `max_features` | | `1.0` on the bagging estimators, `"sqrt"` on `RandomForestClassifier` | each member is allowed more columns, so it is individually stronger and more like its siblings: $\sigma^{2}$ down, $\rho$ up. Lowering it is the random subspace mechanism and trades the same two quantities the other way | tune it as the diversity dial once $M$ has flattened, scoring on held-out folds. Note the two estimators mean different things by the name: the bagging estimators draw one feature subset per member and keep it for that member's whole fit, while the forests redraw a subset at every node |
| features drawn with replacement, `bootstrap_features` | | `False` | a member's feature subset may then contain the same column more than once, which wastes slots in the subset rather than adding information, so the effective number of distinct columns per member falls | leave `False`. It exists for completeness and there is no ordinary reason to set it |

`random_state` changes which rows and which features each member draws, so it changes the fitted ensemble, but it is pinned rather than tuned: fix an integer and see [[Random Seed]]. Without it the same call returns a different ensemble each time, and two scores that differ by less than that run-to-run spread are not a comparison. The seed matters more here than on a single fit, because every member's training set is a draw from it.

`oob_score` is not in the table on purpose. It computes the estimate of the previous section and stores it as `oob_score_`, and it changes nothing about the fitted members or about what `predict` returns, so there is nothing to tune. `warm_start` is likewise excluded: it reuses the members already fitted and adds more, which is a way of walking up the $M$ column above rather than a parameter of its own. `n_jobs` is a speed knob and out by the same rule that keeps it out everywhere.

## Failure modes

- **Setting `bootstrap=False` while leaving `max_samples` at `1.0`.** Every member is then handed the entire training split, and for a deterministic base learner every member is the identical model. The ensemble is one model fitted $M$ times, its variance is $\sigma^{2}$ exactly as the $\rho = 1$ case of the formula says, and nothing raises an error: the fit succeeds, the score comes back, and the only clue is that it equals the single model's. The one thing the library does catch is the out-of-bag request that comes with it, since `oob_score=True` and `bootstrap=False` together raise rather than silently scoring nothing.
- **Tuning on the out-of-bag score and then reporting it.** The estimate is honest about rows the members did not see, and it stops being honest the moment choices are conditioned on it. Search $M$, the depth of the base learner and `max_features` against the out-of-bag score and it has become a validation score, with the optimism that goes with being selected on, which is [[Data Snooping Bias]] reached through a held-out set nobody split off by hand. A [[Testing Set]] still has to be held back.
- **Reading a small ensemble's out-of-bag score as the full ensemble's.** At $M = 10$ it is the score of an ensemble of about four members, computed over the ninety-nine percent of rows that have any out-of-bag prediction at all. Both distortions point the same way, so the estimate is pessimistic exactly where the ensemble is cheapest to fit and the temptation to trust it is greatest.
- **Bagging under [[Class Imbalance]].** A bootstrap resample is drawn from every class at once and therefore preserves the class proportions only in expectation; the variance around that is what hurts a rare class. With $m_{\min}$ minority rows out of $n$, a given bag contains none of them with probability $(1 - m_{\min}/n)^{n} \approx e^{-m_{\min}}$, which is about $0.37$ for a single minority row and about $0.007$ for five. Members fitted on such a bag cannot predict the rare class at all, and a plurality vote over the members buries the ones that can. Rebalancing is [[Resampling]]'s business and has to happen inside each member's fit, not before the resampling.
- **Bagging a classifier that is worse than chance somewhere.** The never-worse guarantee is derived under squared error, where the aggregate is a genuine average, and it does not carry over to a vote. In a region where each member is wrong more often than right, the members vote unanimously for the wrong label and the ensemble is worse there than any single member's expected performance, which is Breiman's point that aggregation turns good classifiers into nearly optimal ones and can turn poor ones into worse ones. The check is that each member clears a [[Baseline Model]] on its own before being enrolled.
- **Bagging to fix a bias problem.** Averaging leaves the shared bias of the members intact, so a linear model bagged a hundred times reproduces approximately the single linear fit and a systematically wrong feature set stays systematically wrong. The [[Learning Curve]] shape that says "more data will not help" says the same thing about more members.
- **Losing the thing that made the base learner worth using.** A single decision tree can be read as a sequence of decisions and shown to somebody; a hundred of them cannot. The ensemble also costs $M$ times the storage and $M$ times the prediction latency, which is the deployment cost [[Ensemble Learning]] weighs against the size of the gain.

## Implementation

scikit-learn 1.6:

```python
from sklearn.ensemble import BaggingClassifier, BaggingRegressor
from sklearn.tree import DecisionTreeClassifier

bag = BaggingClassifier(
    DecisionTreeClassifier(),   # the base learner, refitted per member
    n_estimators=500,
    max_samples=1.0,            # each bag is the size of the training split
    bootstrap=True,             # with replacement: this is what makes it bagging
    oob_score=True,             # score on the rows each member did not see
    random_state=42,
)
bag.fit(X_train, y_train)
bag.oob_score_                  # the out-of-bag estimate, available only after fit
```

The 1.6 signature is `BaggingClassifier(estimator=None, n_estimators=10, *, max_samples=1.0, max_features=1.0, bootstrap=True, bootstrap_features=False, oob_score=False, warm_start=False, n_jobs=None, random_state=None, verbose=0)`, and `BaggingRegressor` takes the same arguments. `estimator=None` means a `DecisionTreeClassifier` or `DecisionTreeRegressor` with default settings, so the unconfigured estimator is already bagged trees. The base learner must support `sample_weight` or accept being refitted on a subset, which every scikit-learn estimator does.

Three details of that signature decide whether the note above applies to a given call.

- `max_samples` and `max_features` are each an integer count or a float fraction of the available rows or columns. The rows are drawn with replacement when `bootstrap=True` and the columns without replacement when `bootstrap_features=False`, which is the default pair and is the derivation above.
- `oob_score=True` is accepted only alongside `bootstrap=True`. Setting it with `bootstrap=False` raises `ValueError` with the message that out-of-bag estimation is only available if `bootstrap=True`, which is the library refusing to compute a score over an empty set.
- The out-of-bag attributes can contain `NaN`. `oob_score_` is the scalar, and `oob_decision_function_` on the classifier or `oob_prediction_` on the regressor holds the per-row out-of-bag output; the documentation states that a data point may never have been left out when `n_estimators` is small, in which case its entry is `NaN`. That is the $0.632^{M}$ arithmetic above appearing as a value in an array.

The random forest estimators are the same mechanism with the per-node feature draw built in, and their defaults differ in ways worth knowing: `RandomForestClassifier(n_estimators=100, max_features="sqrt", bootstrap=True, oob_score=False, max_samples=None)`, against `max_features=1.0` on `RandomForestRegressor`, so the classifier restricts features per split by default and the regressor does not. The `"sqrt"` default arrived in 1.1, replacing `"auto"`, and the jump from ten members to a hundred happened in 0.22. `oob_score` on the forests takes a callable as well as a boolean, scoring the out-of-bag predictions with any `metric(y_true, y_pred)` rather than with accuracy.
