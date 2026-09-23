---
note_kind: method
aliases:
  - boosting
  - AdaBoost
  - adaptive boosting
  - gradient boosting
  - gradient boosted trees
  - gradient boosting machine
  - GBM
  - AdaBoostClassifier
  - GradientBoostingClassifier
  - HistGradientBoostingClassifier
  - XGBoost
up: "[[Ensemble Learning]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Boosting is a family of iterative ensemble methods that fit members in sequence, each one conditioned on what its predecessors got wrong, and combine them into a weighted sum. Every member sees the same rows; what changes between rounds is how much each row is worth, so the later members concentrate on the instances the earlier ones got wrong. The promise the family is named for is that this converts weak learners into strong ones, and that promise is a theorem with a bound attached rather than a description of what usually happens, which is the thing worth getting right about it.

Reach for it when the problem is bias rather than variance. [[Ensemble Learning]] shows that averaging attacks only the variance of the members and leaves any bias they share exactly where it was, so a committee of members that are all systematically wrong in the same direction stays systematically wrong ([[Underfitting]]). Sequential fitting is the mechanism that supplies what averaging cannot: each member is fitted against the residual error of the ensemble so far, so the ensemble's bias is what the next member is built to reduce. That is why the usual base learner here is deliberately weak, a stump or a tree of depth three, where [[Bagging]] wants its members as strong and as unstable as it can get them.

The cost of the same design is that boosting can overfit and bagging essentially cannot. Members here are not independent draws, they are a fitted sequence, so the number of rounds is a capacity parameter: each round adds flexibility aimed at the residual, and past some round the residual it is chasing is noise. Everything in the hyperparameter table exists to slow that down.

## Algorithm

The shape of the family, in the order the steps happen. Written for a weighted-instance scheme, which is AdaBoost's.

1. **Train the first weak model on the original data**, with every instance weighted equally.
2. **Reweight the instances** according to how well that model predicted them, giving the misclassified instances higher weight.
3. **Train the second model on the reweighted data.** The ensemble now consists of the first and second models.
4. **Reweight the instances** again, this time according to how well the *ensemble* predicted them rather than how well the last member alone did.
5. **Train the third model on the reweighted data.**
6. **Repeat** for $n$ iterations.
7. **Form the final strong model as a weighted combination** of the members, each member's weight set by how well it did on the distribution it was handed.

Step 4 is the step that makes this boosting and not a sequence of unrelated fits: the target of the reweighting is the ensemble's error, not the last member's, so a mistake that the ensemble has already recovered from stops being emphasised. Two things follow from the sequence being a sequence. The members cannot be fitted in parallel, since member $t$ needs the weights that member $t-1$ produced. And the base learner has to accept instance weights, which is where this meets [[Loss Reweighting]]: each round is literally a weighted-risk fit, the $\frac{1}{\sum_i w_i}\sum_i w_i \ell(y^{(i)}, \hat{y}^{(i)})$ objective that note owns, with the weights supplied from outside as `sample_weight`. What separates boosting from the members of that family is the argument of the weight. A class-prior weight is a function of the label and a cost weight is a function of the label, so both are fixed before the fit starts; boosting's weight is a function of the ensemble's current error on that instance, so it is recomputed every round, which is the same kind of weight focal loss uses and a different mechanism from either.

### AdaBoost's weights, exactly

The two-class case with labels $y \in \{-1, +1\}$ is where the arithmetic is worth writing down, because every claim the family makes rests on it. Let $D_t(i)$ be the weight on instance $i$ at round $t$, starting from the uniform $D_1(i) = 1/m$.

1. Fit $h_t$ on the weighted data and compute its **weighted error**
   $$\epsilon_t = \sum_{i=1}^{m} D_t(i)\,\mathbb{1}\big[h_t(\mathbf{x}^{(i)}) \ne y^{(i)}\big]$$
2. Set the member's weight in the final vote,
   $$\alpha_t = \frac{1}{2}\ln\frac{1 - \epsilon_t}{\epsilon_t}$$
3. Reweight and renormalize,
   $$D_{t+1}(i) = \frac{D_t(i)\,\exp\!\big(-\alpha_t\, y^{(i)} h_t(\mathbf{x}^{(i)})\big)}{Z_t}$$
   with $Z_t$ the constant that makes $D_{t+1}$ sum to one.
4. Output $H(\mathbf{x}) = \operatorname{sign}\!\left(\sum_{t=1}^{T} \alpha_t h_t(\mathbf{x})\right)$.

Three readings of $\alpha_t$. It is positive exactly when $\epsilon_t < 1/2$, so a member that beats chance is added and one that does not would be subtracted, and it grows without bound as $\epsilon_t \to 0$, so a nearly perfect member dominates the vote. The exponent $-\alpha_t y^{(i)} h_t(\mathbf{x}^{(i)})$ is $-\alpha_t$ on a correct instance and $+\alpha_t$ on a wrong one, so the reweighting multiplies the misclassified instances by $e^{\alpha_t} = \sqrt{(1-\epsilon_t)/\epsilon_t} > 1$ and the correct ones by its reciprocal. And this particular value is not a heuristic: it is what makes the weighted error of $h_t$ under the *next* distribution exactly one half,

$$\sum_{i:\, h_t(\mathbf{x}^{(i)}) \ne y^{(i)}} D_{t+1}(i) = \frac{\epsilon_t \, e^{\alpha_t}}{Z_t} = \frac{\sqrt{\epsilon_t(1-\epsilon_t)}}{2\sqrt{\epsilon_t(1-\epsilon_t)}} = \frac{1}{2}$$

which is to say the reweighting moves the distribution to exactly the point where the member just fitted has no remaining edge, so the next member has nothing to gain by repeating it.

A note on provenance, because the two forms in circulation are not the same expression. Freund and Schapire's paper (*Journal of Computer and System Sciences* 55(1), 1997) writes the update through $\beta_t = \epsilon_t/(1 - \epsilon_t)$, with no normalizing $Z_t$, renormalizing the weight vector into a distribution at the top of each round instead, and gives each member the vote weight $\log(1/\beta_t)$, which is $2\alpha_t$. The $\alpha_t$ and $Z_t$ form above is Schapire and Singer's (*Machine Learning* 37(3), 1999), where $\alpha_t$ is derived as the minimizer of $Z_t$, and it is the form Freund and Schapire's own 1999 tutorial states in pseudocode. The two are the same algorithm: doubling every vote weight and doubling the threshold changes no prediction.

### Why "weak to strong" is a theorem

The claim that boosting converts weak learners into strong ones is Schapire's (*Machine Learning* 5(2), 1990), and its content is an equivalence, not an improvement: a concept class is weakly learnable if and only if it is strongly learnable. Weak learnability there means the learner returns a hypothesis whose error is at most $\frac{1}{2} - \frac{1}{p(n,s)}$ for some polynomial $p$ in the problem size, meaning slightly better than guessing; strong learnability means an error of at most any $\epsilon$ you name, in time polynomial in $1/\epsilon$. The original construction is a recursive majority vote of three hypotheses, each trained on a filtered distribution, and it is the existence proof rather than the algorithm anybody runs.

AdaBoost is what makes the equivalence practical, and the reason is one inequality. Its training error on the $m$ fitted instances is bounded by

$$\frac{1}{m}\Big|\big\{\, i : H(\mathbf{x}^{(i)}) \ne y^{(i)} \,\big\}\Big| \;\le\; \prod_{t=1}^{T} 2\sqrt{\epsilon_t\,(1 - \epsilon_t)}$$

which is Theorem 6 of the 1997 paper, stated there with the $2^{T}$ pulled out front. Write $\epsilon_t = \frac{1}{2} - \gamma_t$, so $\gamma_t$ is the member's edge over chance, and the same bound becomes

$$\prod_{t=1}^{T}\sqrt{1 - 4\gamma_t^{2}} \;\le\; \exp\!\left(-2\sum_{t=1}^{T}\gamma_t^{2}\right)$$

This is what makes the slogan a theorem. Each factor $2\sqrt{\epsilon_t(1-\epsilon_t)}$ is strictly less than one whenever $\epsilon_t \ne 1/2$, so every member that is not exactly at chance shrinks the bound, and if every member holds an edge of at least $\gamma$ then the training error is at most $e^{-2T\gamma^{2}}$, falling exponentially in the number of rounds. Concretely, a weak learner returning $\epsilon_t = 0.3$ every round has a per-round factor of $2\sqrt{0.21} \approx 0.9165$, so the bound is $0.418$ after ten rounds, $0.0128$ after fifty, $1.6 \times 10^{-4}$ after a hundred, and is below $1/60{,}000$ after about 127 rounds, at which point a training set of sixty thousand rows is fitted perfectly. Nothing weaker than "does better than a coin" is assumed anywhere in that.

Two limits of what has just been proved, both easy to overstate. The bound is on the **training** error, so it says the ensemble drives its error on the fitted rows to zero and says nothing directly about [[Generalization]]. And the edge is required at every round on the *reweighted* distribution, which gets harder as the rounds go on, since step 3 above leaves the previous member with no edge at all; a base learner that runs out of edge is where the sequence actually stops.

### Gradient boosting

Friedman's reformulation (*Annals of Statistics* 29(5), 2001) drops the instance weights and replaces them with something more general: fit each new member to the negative gradient of the loss with respect to the current model's predictions. At round $m$, with the ensemble so far written $F_{m-1}$, compute for each instance what he calls the pseudoresponse

$$\tilde{y}_i = -\left[\frac{\partial L\big(y^{(i)}, F(\mathbf{x}^{(i)})\big)}{\partial F(\mathbf{x}^{(i)})}\right]_{F = F_{m-1}}$$

fit the base learner to those $\tilde{y}_i$ by least squares, find the step length along it that minimizes the real loss, and add it to $F_{m-1}$. Under squared error the pseudoresponse is the ordinary residual, which is where the intuition "each tree fits the previous trees' mistakes" comes from, and under any other differentiable loss it is the correct generalization of that. The connection to [[Gradient Descent]] is exact and is the reason for the name: this is a descent step, but taken in the space of functions rather than in a [[Parameter Space]], with the base learner supplying the direction.

The knob that matters most is the one Friedman introduces as regularization, a shrinkage factor $0 < \nu \le 1$ scaling each update:

$$F_m(\mathbf{x}) = F_{m-1}(\mathbf{x}) + \nu\,\rho_m h(\mathbf{x}; \mathbf{a}_m)$$

He reports that decreasing $\nu$ improves accuracy, often dramatically, with a diminishing return below about $0.125$, that it trades directly against the number of rounds, since a smaller $\nu$ pushes the best $M$ higher, and that the practical recipe is to set $M$ as large as the compute allows and then choose $\nu$ so the loss bottoms out near it. His own experiments on real data use $\nu = 0.1$, which is where the libraries' defaults come from. Notably he also reports that he does not fully understand why shrinkage helps as much as it does.

Row subsampling is a separate paper, Friedman's stochastic gradient boosting (*Computational Statistics and Data Analysis* 38(4), 2002), which draws a subsample of size $\tilde{N} = fN$ **without** replacement at each round and fits that round on it alone. The fraction $f$ trades the same two things as everywhere else: a smaller $f$ makes successive rounds differ more, which is randomness of the kind [[Bagging]] exploits, while leaving each round less data and so raising the variance of each member. He notes $f = 1/2$ is roughly equivalent to drawing a bootstrap sample each round, and lands empirically on $0.5 \le f \le 0.8$.

XGBoost (Chen and Guestrin, KDD 2016) is the engineering of the same method plus one change to the objective. It adds a penalty on the tree itself,

$$\Omega(f) = \gamma T + \tfrac{1}{2}\lambda \lVert \mathbf{w} \rVert^{2}$$

for $T$ leaves and leaf weights $\mathbf{w}$, so the number of leaves and the size of their values are both paid for, and it expands the loss to second order in the update, using the Hessian as well as the gradient, which gives closed forms for the optimal leaf value and for the gain of a candidate split. The paper is explicit that the second-order expansion is not its own invention and that with the regularization parameters at zero the objective falls back to ordinary gradient tree boosting. What is its own is the systems work: an approximate split finder using a weighted quantile sketch, so that candidate cut points are weighted by the Hessian rather than counted uniformly, and a sparsity-aware split finder that learns a default direction per node for missing values instead of imputing them.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of rounds, `n_estimators` or `max_iter` | $T$, $M$ | `50` on `AdaBoostClassifier`, `100` on `GradientBoostingClassifier` and `HistGradientBoostingClassifier` | this is a capacity parameter, unlike its bagged namesake. The training error falls toward the bound above while the validation error follows it down and then turns back up, because the residual the later rounds chase stops being signal. Fit time and prediction latency grow linearly | do not search it directly. Set it as high as the compute allows, turn on [[Early Stopping]] against a validation fold, and let the run find the round count. Searching $T$ and the learning rate independently is wasted work, since they trade against each other |
| learning rate, shrinkage | $\nu$, $\eta$ | `1.0` on `AdaBoostClassifier`, `0.1` on both gradient boosting estimators | each round contributes more, so the ensemble reaches a given training error in fewer rounds and overfits in fewer rounds too. Lowering it is the single most reliable accuracy improvement in the family, and it raises the best $T$ in proportion | fix $T$ at the compute budget and lower $\nu$ until the validation loss bottoms out close to the last round. Below about $0.125$ the returns diminish, which is the region the $0.1$ default sits in |
| base learner capacity, `max_depth` or `max_leaf_nodes` | | `1` implicitly on `AdaBoostClassifier`, which uses a depth-one stump; `max_depth=3` on `GradientBoostingClassifier`; `max_leaf_nodes=31` on `HistGradientBoostingClassifier` | each member can express more, including interactions of more features, so the ensemble reaches a lower training error per round and needs fewer rounds, and it overfits sooner. A deep member also makes the sequence less stable, since one round can absorb a large part of the residual | keep members weak: this is the parameter the family's premise is about. Tune it over a small range, $1$ to about $8$ in depth, jointly with $\nu$ and $T$, and treat a best value at the top of the range as a sign the problem wanted [[Bagging]] instead |
| row subsample fraction, `subsample` | $f$ | `1.0` on `GradientBoostingClassifier`, meaning off | below $1.0$ each round sees fewer rows, so successive rounds differ more and the ensemble is regularized, at the cost of a noisier member per round. At `1.0` the procedure is deterministic given the data | leave off unless the fit is overfitting with $\nu$ already low, then try the $0.5$ to $0.8$ range. A value of $0.5$ is roughly as much randomness as a bootstrap resample supplies |
| leaf penalty, `l2_regularization` | $\lambda$ | `0.0` on `HistGradientBoostingClassifier`, meaning off | leaf values are shrunk toward zero in proportion to how little curvature supports them, so leaves resting on few instances contribute less and the member is smoother | raise it on a log scale when the members are deep or the leaves are small, scoring on held-out folds. It is an alternative route to the same place as lowering $\nu$, so tune one at a time |
| early stopping switch, `early_stopping` | | `'auto'` on `HistGradientBoostingClassifier`, which enables it above ten thousand rows; `n_iter_no_change=None` on `GradientBoostingClassifier`, which disables it | turning it on changes what the fit returns, since the ensemble stops at the round where a held-out score stopped improving rather than at $T$. The companion `validation_fraction` decides how many rows are taken out of the fit to judge that | turn it on and leave the round count generous, which is the cheap way to tune $T$. Note the two estimators disagree by default, so the same nominal $T$ means different things to them |

`random_state` is pinned rather than tuned: it fixes the row subsample of each round, the tie-breaking in the tree fits, and the validation split that early stopping reads, so it changes the fitted ensemble without having a direction to tune in. Fix an integer, see [[Random Seed]]. Its effect here is larger than on a bagged ensemble, because a different first-round subsample changes every subsequent round's residual rather than one member out of $M$.

Two things that look like parameters and are not. `max_bins` on the histogram estimator decides the resolution of the feature binning, which changes speed and memory far more than it changes the fit, and it is capped at $255$. `n_jobs` and `verbose` are out by the usual rule.

## Failure modes

- **Treating the round count the way a bagged ensemble's is treated.** Adding members to a [[Bagging]] ensemble cannot make it worse; adding rounds to a boosted one can and does, because each round is fitted against the ensemble's current residual and eventually that residual is noise. A boosted model with the round count set generously and no [[Early Stopping]] is a model that was left training past its best epoch, and the training loss gives no sign of it: it is still falling, exactly as the bound says it should.
- **Boosting on a target with mislabelled instances.** The reweighting is specifically designed to put more weight on whatever the ensemble keeps getting wrong, and a wrongly labelled instance is unlearnable, so its weight grows every round. Under AdaBoost the multiplier is $e^{\alpha_t}$ per round, so the weight on a permanently misclassified instance compounds and the later members spend their capacity on it. Noise that a bagged ensemble averages away is noise boosting chases.
- **A base learner that cannot use the weights.** The whole scheme is a sequence of weighted fits, and an estimator that silently ignores `sample_weight` turns it into a sequence of identical fits on the same rows, with the member weights $\alpha_t$ all equal and the ensemble equal to one model. The scikit-learn estimators require weight support of the base learner for exactly this reason.
- **A base learner that is too strong.** Hand AdaBoost a fully grown tree and the first member reaches $\epsilon_1 \approx 0$, which sends $\alpha_1 = \frac{1}{2}\ln\frac{1-\epsilon_1}{\epsilon_1}$ toward infinity and makes the ensemble that one member. The bound is technically satisfied and nothing has been gained: the family's premise is a *weak* learner with an edge, and a member that memorizes its weighted sample has no edge to hand the next round.
- **Reading the training error as progress.** The bound above is a guarantee about the fitted rows and about nothing else, so a boosted ensemble driving its training error to zero is doing what was proved rather than doing well. The only reading that means anything is a held-out one, which is [[Cross-Validation]]'s job here as much as anywhere.
- **Expecting parallelism.** Round $t$ cannot start before round $t-1$ finishes, so the wall-clock time is the number of rounds times the cost of a fit, and the `n_jobs` argument on these estimators parallelizes work inside a round rather than across rounds. A [[Bagging]] ensemble of the same size fits in a fraction of the time on the same hardware.
- **Boosting to fix a variance problem.** The mechanism adds capacity aimed at the residual, so applying it to a base learner that is already overfitting ([[Overfitting]]) makes the fit worse in the direction it was already failing. The choice between the two families is a reading of which term is large, and the [[Learning Curve]] is what supplies it.

## Implementation

scikit-learn 1.6 carries three of these, and they are not interchangeable.

```python
from sklearn.ensemble import (AdaBoostClassifier, GradientBoostingClassifier,
                              HistGradientBoostingClassifier)
from sklearn.tree import DecisionTreeClassifier

# AdaBoost: reweighted instances, a depth-one stump by default
ada = AdaBoostClassifier(
    DecisionTreeClassifier(max_depth=1),
    n_estimators=200,
    learning_rate=0.5,
    random_state=42,
)

# Gradient boosting: each tree fitted to the negative gradient of the loss
gbrt = GradientBoostingClassifier(
    learning_rate=0.05,
    n_estimators=500,
    max_depth=3,
    subsample=0.8,          # stochastic gradient boosting
    n_iter_no_change=20,    # early stopping, off by default on this estimator
    validation_fraction=0.1,
    random_state=42,
)

# Histogram gradient boosting: binned features, leaf-wise growth, NaN handled natively
hgb = HistGradientBoostingClassifier(
    learning_rate=0.05,
    max_iter=1000,
    max_leaf_nodes=31,
    l2_regularization=1.0,
    early_stopping=True,    # 'auto' by default, which turns on above 10,000 rows
    random_state=42,
)
```

The 1.6 signatures, which is where the differences live:

- `AdaBoostClassifier(estimator=None, *, n_estimators=50, learning_rate=1.0, algorithm='deprecated', random_state=None)`. `estimator=None` means `DecisionTreeClassifier(max_depth=1)`, a stump, and the base learner is required to support `sample_weight`. The `algorithm` argument is being retired: `'SAMME.R'` was removed in 1.6, leaving `'SAMME'` as the only implementation, and the parameter itself is deprecated in 1.6 for removal in 1.8, with a runtime `FutureWarning` saying it has no effect. Do not pass it.
- `GradientBoostingClassifier(*, loss='log_loss', learning_rate=0.1, n_estimators=100, subsample=1.0, criterion='friedman_mse', max_depth=3, ..., validation_fraction=0.1, n_iter_no_change=None, tol=0.0001, ...)`. `loss='exponential'` recovers AdaBoost's objective, which is the historical link between the two made available as an argument. Early stopping is off by default here, since `n_iter_no_change` is `None`.
- `HistGradientBoostingClassifier(loss='log_loss', *, learning_rate=0.1, max_iter=100, max_leaf_nodes=31, max_depth=None, min_samples_leaf=20, l2_regularization=0.0, max_features=1.0, max_bins=255, categorical_features='from_dtype', ..., early_stopping='auto', scoring='loss', validation_fraction=0.1, n_iter_no_change=10, tol=1e-07, ...)`. The documentation states that `'auto'` enables early stopping when the sample size is larger than ten thousand, which means the same call is early-stopped on a large file and not on a small one. Two further properties make this the one to reach for first on tabular data: it has native support for missing values, learning at each split whether missing rows go left or right by potential gain rather than requiring [[Missing Value Imputation]], and with `categorical_features='from_dtype'`, which became the default in 1.6, it reads pandas or polars columns of dtype `category` as categorical without [[One-Hot Encoding]]. The documentation credits LightGBM as the inspiration for the implementation.

The names differ in a way that causes real mistakes: the round count is `n_estimators` on the first two and `max_iter` on the histogram estimator, and a `GridSearchCV` parameter grid written for one silently fails to reach the other. XGBoost, LightGBM and CatBoost are separate packages on their own release trains, each with a scikit-learn-compatible wrapper and its own argument names again (`eta` for the learning rate, `num_boost_round` for the round count in XGBoost's native interface).
