---
note_kind: method
aliases:
  - resampling
  - oversampling
  - undersampling
  - random oversampling
  - random undersampling
  - SMOTE
  - synthetic minority oversampling technique
  - Tomek links
  - NearMiss
  - near miss
  - one-sided selection
  - two-phase learning
  - dynamic sampling
up: "[[Class Imbalance]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Resampling changes the class proportions of the training set before anything is fitted, by adding instances of the rare classes, removing instances of the common ones, or both. The point is not to get more information out of the data: it is that a loss summed over rows pays the optimizer in proportion to how many rows each class contributes, so under [[Class Imbalance]] the cheapest descent direction is the one that serves the majority. Changing the counts changes the payoff.

There are two ways to change that payoff and they are alternatives, not stages. Resampling alters the data the optimizer sees and leaves the objective alone. [[Loss Reweighting]] leaves every row where it is and alters what each row is worth in the objective. Reach for resampling when the model or the library gives you no usable weight vector, when the majority class is large enough that discarding part of it costs nothing you can measure, or when you want a synthetic operator such as SMOTE to fill in the region between rare instances rather than merely to shout louder about the ones you have. Reach for reweighting when every instance is expensive, when the fit is already at the edge of its memory or time budget, or when you want the training distribution to stay exactly as it was collected so the fitted probabilities stay calibrated against it. For a plain sum-of-losses objective the two coincide in one special case: duplicating every minority row $c$ times gives the same total loss as leaving them in place with weight $c$, so random oversampling by an integer factor **is** a reweighting, written into the data instead of the objective. Once the optimizer is stochastic, or regularization touches the parameters rather than the rows, or early stopping decides when to halt, the two stop agreeing, because the number of gradient steps each class buys is no longer the same thing as the weight it carries.

Against its nearest-looking neighbour: [[Data Augmentation]] grows the training set to teach an invariance, by emitting a transformed copy of an instance that a human would still give the same label. Resampling changes the class mix to change what the optimizer is rewarded for, and it teaches nothing. SMOTE belongs to both families on the surface, since it does fabricate rows that were never collected, and it is owned here, because its purpose is the class proportions: it interpolates between two instances of one class rather than applying a known label-preserving transformation to one of them, and nobody runs it on a balanced target.

## Algorithm

Write the training split as $D_{\text{train}}$ with $m_k$ instances in class $k$, the largest count as $m_{\max}$ and the smallest as $m_{\min}$. The target ratio is

$$\alpha = \frac{m_{\min}'}{m_{\max}'}$$

the minority count over the majority count **after** resampling, so $\alpha = 1$ is full balance and $\alpha$ equal to the starting $m_{\min}/m_{\max} = 1/\text{IR}$, the reciprocal of the imbalance ratio the data arrived with, is a no-op.

1. **Split first.** Hold out the [[Testing Set]] and fix the [[Cross-Validation]] folds on the original data. Nothing has been duplicated or interpolated yet at this point.
2. **Fix $\alpha$.** Full balance is the default everywhere and is not automatically the best point; it is one end of a range you can search.
3. **Apply the operator to the training rows only.** One of the families below.
4. **Fit on the resampled training set.**
5. **Score on the untouched held-out data**, which still carries the class mix the model will meet.

### Adding minority instances

**Random oversampling.** Draw from the minority class with replacement until $m_{\min}' = \alpha \, m_{\max}$. Nothing new enters the set: the empirical distribution of the minority class is exactly what it was, only heavier, so every duplicated row raises the weight of a point the model has already seen and pulls the decision boundary toward memorising it. That is the overfitting risk in one sentence, and it is worse for a high-capacity model than for a linear one.

**SMOTE** (Chawla, Bowyer, Hall and Kegelmeyer, *JAIR* 16, 2002) fabricates points instead of copying them. For a minority instance $\mathbf{x}$, find its $k$ nearest neighbours **among the minority class only**, pick one of them at random as $\mathbf{x}^{nn}$, and emit

$$\mathbf{x}_{\text{new}} = \mathbf{x} + \lambda \big( \mathbf{x}^{nn} - \mathbf{x} \big), \qquad \lambda \sim U(0,1)$$

That is the whole method. The new point lies on the line segment between two real minority instances, which is why the authors describe the effect as making the minority decision region more general rather than more populated. The label is copied across; no target is ever interpolated. One implementation detail is worth knowing because the paper and the code differ on it: the published pseudo-code draws `gap` inside the loop over attributes, which gives an independent $\lambda$ per coordinate and places the point somewhere in the axis-aligned box spanned by $\mathbf{x}$ and $\mathbf{x}^{nn}$ rather than on the segment joining them, while `imbalanced-learn` draws one $\lambda$ per synthetic point and broadcasts it across features, which is the segment the prose describes.

### Removing majority instances

**Random undersampling.** Discard majority instances at random until $m_{\max}' = m_{\min}/\alpha$. It is free of the duplication problem and buys a smaller, faster fit, and it pays for that by throwing away real measurements, which is unaffordable when the majority class is itself small.

**Tomek links** (Tomek 1976, *Two Modifications of CNN*) drop a chosen few instead of a random many. A pair $(\mathbf{x}_i, \mathbf{x}_j)$ with $y_i \ne y_j$ is a Tomek link when no third instance $\mathbf{z}$ in the set is closer to either of them than they are to each other,

$$d(\mathbf{x}_i, \mathbf{x}_j) < d(\mathbf{x}_i, \mathbf{z}) \quad \text{and} \quad d(\mathbf{x}_i, \mathbf{x}_j) < d(\mathbf{x}_j, \mathbf{z}) \qquad \text{for every } \mathbf{z} \ne \mathbf{x}_i, \mathbf{x}_j$$

which is to say the two are mutual nearest neighbours across the class boundary. Such a pair is either a mislabelling or a genuine borderline case, and in both readings one of the two sits where the boundary has to pass. Removing the majority member of every link widens the margin and cleans the frontier; removing both members is the alternative setting and is a cleaning operation rather than a balancing one. Notice that the number removed is whatever the geometry yields, so this does not reach a requested $\alpha$ on its own.

**NearMiss** (Mani and Zhang 2003) keeps the majority instances a distance criterion selects, in three variants that differ in which distances are averaged. Version 1 keeps the majority instances whose average distance to their $N$ **closest** minority instances is smallest, so it keeps the ones crowding the boundary. Version 2 keeps those whose average distance to the $N$ **farthest** minority instances is smallest, a global criterion that prefers instances near the minority class as a whole rather than near its edge. Version 3 runs in two stages: first keep, for each minority instance, its $M$ nearest majority neighbours, which guarantees every minority instance stays surrounded, then from that pool keep the ones whose average distance to their $N$ nearest minority instances is **largest**. The three are not points on a scale, and they select visibly different subsets.

**One-sided selection** (Kubat and Matwin, ICML 1997) composes two steps. First the condensed nearest neighbour rule: seed a set with every minority instance plus one majority instance, classify the rest with 1-nearest-neighbour against that set, and move each misclassified instance into it, so what survives is the subset that reproduces the 1-NN decisions and the redundant interior of the majority class is gone. Then remove the majority member of every Tomek link, which strips the borderline and noisy instances the condensation kept. The "one-sided" part is that both steps are allowed to delete majority instances only.

### Scheduling the resampling against training

**Two-phase learning** (Lee, Park and Kim, ICIP 2016) splits the fit rather than the data: train first on a resampled, less imbalanced set so the rare classes get a signal at all, then fine-tune on the original distribution so the fitted priors go back to matching the world.

**Dynamic sampling** (Pouyanfar et al., IEEE MIPR 2018) makes the ratio a function of training progress instead of a constant. Between epochs, read the per-class performance and oversample the classes doing badly while undersampling the ones doing well, so the mix keeps moving toward whatever the model has not learned yet.

### The rule: a model is never evaluated on resampled data

Resampling is applied to the training split, after the split, and to nothing else. The [[Testing Set]] and every validation fold keep the class mix the deployment data has, because a score computed on a rebalanced evaluation set is a score on a distribution nobody will ever send you: the priors $\pi_k$ it was measured under are ones you manufactured, and any measure that depends on them, [[Accuracy]] and [[Precision]] above all, moves with them. Worse, a duplicated or interpolated row derived from a training instance and landing in the held-out part makes the model's own training data part of its exam, which is the same hazard as leaking across a split and is invisible afterwards. This is the single rule in the note that a wrong answer cannot be recovered from, because the number that tells you something went wrong is the number that is wrong.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| target ratio, `sampling_strategy` | $\alpha$ | `'auto'`, which is full balance: for an over-sampler every non-majority class is raised to $m_{\max}$, for an under-sampler every non-minority class is cut to $m_{\min}$, either way $\alpha = 1$ | the training mix moves further from the collected one, so the model shifts more probability mass to the rare class: recall rises and precision falls, and the fitted scores drift further from calibration against the deployment priors | search it rather than assuming $1$. Sweep a few values between the starting $1/\text{IR}$ and $1$, scoring each on unresampled held-out folds with a measure that survives imbalance, and keep the operating point the problem actually wants |
| SMOTE neighbourhood | $k$ | $5$, the value the original paper's implementation used and the `k_neighbors` default | each new point is drawn toward a wider set of minority instances, so the synthetic cloud spreads and smooths; large $k$ reaches across sparse regions the class does not really occupy, small $k$ clusters new points tightly around the originals and approaches plain duplication | raise it when the minority class is dense and low dimensional, lower it when minority instances are few or scattered. It is bounded above by $m_{\min} - 1$, and a class with fewer instances than $k+1$ cannot be SMOTEd at all |
| NearMiss variant | `version` | $1$ | not an ordered scale. Moving $1 \to 2$ swaps the $N$ closest minority instances for the $N$ farthest in the averaged distance, trading a boundary-local criterion for a global one; $3$ adds the guarantee that every minority instance keeps its $M$ nearest majority neighbours and then inverts the criterion to keep the *most* distant survivors | pick by inspecting what each keeps on your data rather than by reputation. Version 1 is the default and the most aggressive about the boundary; version 3 is the one to try when version 1 leaves the minority class surrounded by nothing |
| resampling schedule | | once, before the fit | moving from one pass to per-epoch resampling means the model never sees the same resampled set twice, which regularizes harder and makes the training set non-reproducible without a fixed [[Random Seed]]; making the mix respond to per-class performance is dynamic sampling, and it adds a feedback loop that can oscillate | once up front for a one-shot fit, which is also the only form the `imbalanced-learn` estimators offer. Per-epoch or performance-driven schedules belong to iterative training and are written by hand |

Pinned rather than tuned: `random_state`, which fixes which rows are duplicated, dropped or interpolated and so changes the resampled set that comes back, but has no direction to tune in (see [[Random Seed]]); `replacement` on `RandomUnderSampler`, which decides whether the retained majority rows are drawn with repetition and is left `False`; and the choice of distance $d$ in every neighbourhood-based operator above, which is Euclidean throughout and is a modelling decision about the feature space, not a dial.

## Failure modes

- **Scoring the model on the resampled set.** Balance the training data, forget to score on the original mix, and every measure that depends on the priors reports a model that does not exist. The floor a constant predictor takes is $\max_k \pi_k$, so rebalancing a 1:99 target to 1:1 moves the [[Accuracy]] floor from $0.99$ to $0.50$: a model that got worse can report a number that looks better, and there is no trace of it in the output.
- **Resampling before the split, or before the cross-validation folds.** This is the same leak as augmenting before a split and is more insidious, because the duplicate is not merely similar to a training row, it *is* that row. Run `SMOTE` over the whole set and then `KFold`, and interpolations of held-out instances sit in the training folds while their parents are scored; a nearest-neighbour model will come close to perfect and have learned nothing. Random oversampling fails the same way with exact copies on both sides of the boundary. The fix is structural rather than attentive: resample inside the fold, which is what `imblearn.pipeline.Pipeline` exists to guarantee.
- **SMOTE interpolating across the class boundary or into empty space.** The segment between two minority instances is only minority territory if the class is locally convex there. Where the rare class comes in separated clumps, the line joining two of them runs straight through majority territory, and the synthetic points planted along it are mislabelled by construction; where the minority instance is an outlier, the segment reaches into a region no real instance occupies. Blagus and Lusa (*BMC Bioinformatics* 14:106, 2013) measured the consequence on high-dimensional class-imbalanced data: SMOTE did not remove the bias toward the majority class for most classifiers and was generally beaten by plain random undersampling, the one exception being $k$-nearest-neighbour classifiers on Euclidean distance, and then only when variable selection ran first.
- **Trusting neighbourhood distances in a high-dimensional feature space.** SMOTE, Tomek links, NearMiss and one-sided selection are all nearest-neighbour computations, and as the dimension grows the distances from a point to its nearest and its farthest neighbour converge toward each other, so "nearest" stops distinguishing anything and the selection becomes close to arbitrary. The cost is paid twice: the pairwise distances are expensive to compute on a large majority class, and what they buy degrades exactly as the feature space that made them expensive gets wider.
- **Undersampling away the instances that defined the boundary.** Random undersampling has no idea which majority rows matter, so it discards support instances and interior redundancy at the same rate, and the boundary moves for no reason connected to the data. The informed variants attack this and overcorrect in the other direction: NearMiss version 1 keeps precisely the majority instances closest to the minority class, so a single mislabelled row near the frontier is exactly the row it is most likely to keep.
- **Rebalancing a rare class that is rare because it is genuinely underdescribed.** If the minority class has a handful of instances, resampling multiplies those instances and not the information in them, and the result looks like a balanced training set while remaining a coverage problem ([[Few-Shot Learning]]). Nothing in the resampled counts reveals this, which is why the check is $m_{\min}$ itself and not $\alpha$.

## Implementation

**scikit-learn 1.6 has no resampling estimator for this.** Its transformers map rows one for one and never change the number of rows, and `sklearn.pipeline.Pipeline` has no stage at which the sample count could change, so none of the family above can be expressed in it. The implementations live in **`imbalanced-learn`, which is a separate package on its own release train**, distributed as `imbalanced-learn` on PyPI and `imblearn` on import, maintained under `scikit-learn-contrib` rather than in scikit-learn. It is described often enough as "scikit-learn's resampling module" that the distinction is worth stating outright: installing scikit-learn does not install it, and its version number is its own.

Everything below is checked against **imbalanced-learn 0.14.2**, whose own dependency floor is scikit-learn 1.4.2. The version pinned here is imbalanced-learn's, not scikit-learn's.

```python
from imblearn.over_sampling import RandomOverSampler, SMOTE
from imblearn.under_sampling import (
    RandomUnderSampler, TomekLinks, NearMiss, OneSidedSelection)

RandomOverSampler(sampling_strategy="auto", random_state=42, shrinkage=None)
SMOTE(sampling_strategy="auto", random_state=42, k_neighbors=5)
RandomUnderSampler(sampling_strategy="auto", random_state=42, replacement=False)
TomekLinks(sampling_strategy="auto")
NearMiss(sampling_strategy="auto", version=1, n_neighbors=3, n_neighbors_ver3=3)
OneSidedSelection(sampling_strategy="auto", random_state=42,
                  n_neighbors=None, n_seeds_S=1)
```

Each exposes `fit_resample(X, y)` and returns a new `X_resampled, y_resampled` rather than a transformed array of the same length, which is the API difference that keeps them out of scikit-learn. `sampling_strategy` takes `'auto'`, a string naming which classes to touch (`'minority'`, `'not majority'`, `'not minority'`, `'all'`), a float, or a dict of per-class target counts. A float is $\alpha$ above, the minority count over the majority count after resampling, and it is accepted for binary targets only; multiclass raises.

The safe way to use any of them is `imblearn.pipeline.Pipeline`, which is not scikit-learn's:

```python
from imblearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score, StratifiedKFold

pipe = Pipeline([
    ("scale", StandardScaler()),
    ("resample", SMOTE(sampling_strategy=0.5, k_neighbors=5, random_state=42)),
    ("clf", LogisticRegression(max_iter=1000)),
])

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
cross_val_score(pipe, X, y, cv=cv, scoring="average_precision")
```

The difference that matters is when the sampler runs. A resampling stage inside this pipeline is applied during `fit` and skipped during `predict`, `predict_proba` and `score`, so each training fold is rebalanced and each validation fold is scored at the class mix it was drawn with, which is the rule above enforced by the plumbing rather than by remembering. The documentation is candid that this breaks a scikit-learn contract: `fit_transform(X, y)` is no longer equivalent to `fit(X, y).transform(X)`, because the resampling happens in the first and not the second. That inconsistency is the feature. Putting a sampler in scikit-learn's own `Pipeline` is not an option at all, since it rejects a step that changes the row count.

`Pipeline` also fixes the leak by construction, which is why the resampler belongs inside it and never in a line of its own above the `cross_val_score` call.
