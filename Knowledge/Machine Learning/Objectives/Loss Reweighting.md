---
note_kind: method
aliases:
  - loss reweighting
  - sample weights
  - sample weighting
  - sample_weight
  - class weights
  - class_weight
  - cost-sensitive learning
  - cost sensitive learning
  - cost matrix
  - class-balanced loss
  - class balanced loss
  - focal loss
up: "[[Cost Function]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Loss reweighting multiplies each instance's contribution to the [[Cost Function]] by a weight, so that the optimizer's incentives stop following the class counts. Nothing about the data changes: the same rows are fitted in the same order, and only what each row is worth to the objective moves. Three named techniques live here, and they are one technique with three answers to a single question, what the weight is a function of. Weight by the class prior and you get the class-balanced loss. Weight by a cost you supply from the problem domain and you get cost-sensitive learning. Weight by the model's own current confidence and you get focal loss. Everything else they share, including the arithmetic below.

Reach for it when the objective is the thing that is wrong. Under [[Class Imbalance]] a model that always answers with the majority is genuinely minimizing an error that counts every instance the same, so the fix is either to stop counting them the same or to stop presenting them in those proportions. Those are the two families and the trade between them is clean: **reweighting keeps the training distribution intact and changes the objective, [[Resampling]] keeps the objective and changes the distribution.** Reweighting therefore never discards a real instance and never duplicates one, which matters when the rare class is small enough that undersampling would throw away most of the majority evidence. Resampling in exchange needs no support from the estimator, so it is the route left when the estimator has no weighting argument at all.

Two different things in this area are both called a weight, and the distinction is worth stating once and plainly. **A weight here decides how much a drawn instance counts in the objective. A weight that decides how likely an instance is to be drawn in the first place is [[Weighted Sampling]].** They enter at different stages: the sampling weight settles what is in the batch, and the loss weight settles what the batch is worth once it has been assembled. The two can be set to mimic each other in expectation, but they are not the same operation, and running both at full strength corrects twice.

## Algorithm or formula

Every member of the family starts from the same object. Take the unweighted risk of [[Cost Function]], attach a non-negative weight $w_i$ to each instance, and normalize by the weight total rather than by the instance count:

$$J(\boldsymbol\theta) = \frac{1}{\sum_{i=1}^{m} w_i}\sum_{i=1}^{m} w_i \, \ell\big(y^{(i)}, \hat{y}^{(i)}\big)$$

Setting every $w_i = 1$ recovers the plain average, so the unweighted objective is the case $w_i \equiv 1$ and not a different formula. The normalization is what keeps the number on the same scale as the unweighted loss; drop it and doubling every weight doubles the reported loss without changing the argmin. What follows is three ways of setting $w_i$.

### By class prior

Let $k(i)$ be the class of instance $i$ and $m_k$ the number of instances in class $k$, across $K$ classes. The raw form is the one the inverse-count rule asks for,

$$w_i \propto \frac{1}{m_{k(i)}}$$

Fix the constant by requiring the weights to average to one, which gives

$$w_i = \frac{m}{K \, m_{k(i)}}, \qquad \sum_{i=1}^{m} w_i = \sum_{k=1}^{K} m_k \cdot \frac{m}{K \, m_k} = m$$

and that is exactly what scikit-learn's `class_weight="balanced"` computes. [[Class Imbalance]] already states this formula where it belongs, as the condition under which the correction is needed; this note owns the family of corrections, that note owns the condition. The effect on the objective is that every class contributes the same total weight $m/K$ regardless of how many rows carry it, so a class holding one row in a thousand is worth as much in aggregate as a class holding half the data.

Cui, Jia, Lin, Song and Belongie (CVPR 2019) argue that the raw count is the wrong denominator, and their refinement is the paper the term **class-balanced loss** names. Their claim is that samples of a class overlap: each new example of a class covers ground that earlier examples of the same class already cover, so the information a class actually supplies grows sublinearly in its count. They formalize that as the **effective number** of samples,

$$E_{n_k} = \frac{1 - \beta^{n_k}}{1 - \beta}, \qquad \beta \in [0, 1)$$

with $\beta = (N-1)/N$ for $N$ the volume of all possible data in that class's region of feature space, and weight inversely to it:

$$w_k \propto \frac{1}{E_{n_k}} = \frac{1 - \beta}{1 - \beta^{n_k}}$$

normalized so that $\sum_{k=1}^{K} w_k = K$, which is the same "average to one" convention as above. The two ends of the $\beta$ range are the two positions the family already held. At $\beta = 0$ the weight is $1$ for every class, which is no reweighting at all. As $\beta \to 1$ the ratio tends to $1/n_k$ by L'Hôpital, which is the plain inverse-count rule. So $\beta$ interpolates between doing nothing and inverse frequency, and the paper's experiments search $\beta \in \{0.9, 0.99, 0.999, 0.9999\}$ rather than deriving it.

### By supplied cost

Cost-sensitive learning drops the assumption that all mistakes are worth the same amount and replaces it with a number per kind of mistake. Write $C_{ij}$ for the cost of predicting $j$ when the truth is $i$. The decision rule is then to answer with the class of least expected cost rather than the class of greatest probability,

$$\hat{y}(\mathbf{x}) = \arg\min_{j} \sum_{i} P(i \mid \mathbf{x}) \, C_{ij}$$

which is Elkan's statement of the problem in *The Foundations of Cost-Sensitive Learning* (IJCAI 2001), written there with the transposed convention that the row indexes the prediction. Two consequences of that rule are worth carrying.

$C_{ii}$ need not be zero. A correct decision can carry a cost, as with the clerical work of a correctly refused transaction, or a benefit, which is a negative cost, as with the fee earned on a correctly approved one. Elkan's worked case is a credit-card benefit matrix whose correct-approval entry is a positive fraction of the transaction amount, and his argument is that all four entries have to be measured against one fixed baseline or the matrix is incoherent. What survives the arithmetic is less than the four numbers suggest: adding a constant to the matrix and scaling it by a positive constant both leave the argmin alone, so a two-class cost matrix has essentially one degree of freedom as far as decisions go.

That one degree of freedom is a threshold. In the two-class case the optimal prediction is the positive class exactly when $p = P(y = 1 \mid \mathbf{x}) \ge p^{*}$, with

$$p^{*} = \frac{C_{\text{FP}} - C_{\text{TN}}}{C_{\text{FP}} - C_{\text{TN}} + C_{\text{FN}} - C_{\text{TP}}}$$

and with the diagonal set to zero this collapses to $p^{*} = C_{\text{FP}} / (C_{\text{FP}} + C_{\text{FN}})$. A false negative twelve times as expensive as a false alarm puts the cut at $p^{*} = 1/13 \approx 0.0769$, not at $0.5$. Elkan's own recommendation follows from this: fit the model on the data as given, then set the threshold, rather than distorting the fit and hoping the default cut comes out right.

The same result quantifies the reweighting-versus-resampling trade stated above. Elkan's Theorem 1 says that to make a target threshold $p^{*}$ behave like a learner's built-in threshold $p_0$, the number of negative examples should be multiplied by

$$\frac{p^{*}}{1 - p^{*}} \cdot \frac{1 - p_0}{p_0}$$

which at $p_0 = 0.5$ and a zero diagonal is the factor $C_{\text{FP}} / C_{\text{FN}}$. That is the exchange rate between the two families: a cost ratio in the objective and a count ratio in the data buy the same decision boundary. The difficulty that remains is the one no formula removes. The costs come from the problem domain and not from the data, so nothing in the dataset can be interrogated to produce them, and there is no cross-validation that scores a cost matrix. Expected cost is a [[Performance Measure]] like any other, and an unusual one in that its definition is supplied from outside rather than computed from what is on disk.

### By predicted difficulty

Lin, Goyal, Girshick, He and Dollár (ICCV 2017) set the weight from the model's own output instead. Let $p_t$ be the probability the model currently assigns to the true class, which in the two-class case is $p$ when $y = 1$ and $1 - p$ otherwise. Ordinary cross-entropy is $-\log(p_t)$; focal loss attaches a modulating factor to it:

$$\mathrm{FL}(p_t) = -(1 - p_t)^{\gamma}\log(p_t)$$

and the $\alpha$-balanced variant the paper actually uses in its experiments multiplies by a per-class constant $\alpha_t$:

$$\mathrm{FL}(p_t) = -\alpha_t(1 - p_t)^{\gamma}\log(p_t)$$

At $\gamma = 0$ the factor is $1$ and this is ordinary cross-entropy exactly. As $\gamma$ grows the factor collapses on anything already classified well: an instance at $p_t = 0.9$ with $\gamma = 2$ contributes $(1 - 0.9)^2 = 0.01$ of its usual loss, a hundredfold reduction, while one at $p_t = 0.5$ contributes $0.25$, a fourfold reduction. The ratio between those two reductions is what the gradient sees, so the sum is dominated by the instances the model is still getting wrong.

**What separates this from the first two is the argument of the weight.** The class-prior weight is a function of the label and the cost weight is a function of the label, so both are fixed before training starts and neither moves afterwards. The focal weight is a function of the model's current prediction, so it is recomputed every step and changes as the fit improves: an instance that is hard in epoch one and easy in epoch twenty is weighted heavily and then lightly without anything being reconfigured. It therefore reweights by difficulty and not by class. That it helps under imbalance is a consequence rather than a definition, since a rare class is usually where the hard instances are, and the two can come apart. The $\alpha_t$ term exists precisely because the focusing factor alone does not do the class balancing, and Cui et al.'s class-balanced focal loss is the composition of the first mechanism with this one, the effective-number weight multiplying the focal term.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| focusing parameter | $\gamma$ | $2$ in Lin et al., and $2.0$ in both `torchvision.ops.sigmoid_focal_loss` and Keras 3's focal losses | the down-weighting of confident correct instances steepens, so more of the gradient comes from fewer, harder rows; at $\gamma = 0$ the loss is ordinary cross-entropy and above roughly $5$ the effective training set is small enough that label noise dominates | search the small grid the paper searches, $\{0, 0.5, 1, 2, 5\}$, on a held-out split, and check the loss stays finite; raise it only while the rare-class [[Recall]] is still improving |
| class balance term | $\alpha$ | $0.25$ for the positive class in Lin et al. and in both library implementations | the positive class's share of the loss rises; Lin et al. report that $\alpha = 0.5$ costs $0.4$ AP against $0.25$, so the surface is flat rather than sharp | tune jointly with $\gamma$ and not separately, since the best $\alpha$ falls as $\gamma$ rises: the focusing factor is already suppressing the easy negatives that $\alpha$ was introduced to suppress |
| effective-number decay | $\beta$ | none, the reweighting is opt-in | the weights move from uniform toward inverse frequency: $\beta = 0$ is no reweighting, $\beta \to 1$ gives $w_k \propto 1/n_k$, and the useful settings are crowded near $1$ | search $\{0.9, 0.99, 0.999, 0.9999\}$, the grid Cui et al. search; finer-grained targets take smaller $\beta$, since classes that overlap less gain less from the discount |
| cost matrix entries | $C_{ij}$ | none, the plain objective is $C_{ij} = 1$ off the diagonal and $0$ on it | raising one off-diagonal entry pushes the decision boundary away from that mistake, and by the threshold formula only the ratios matter, so scaling the whole matrix changes nothing | not tuned on the data at all. Elicit each entry from the people who bear the cost, in one unit, against one baseline, then read the threshold $p^{*}$ off it and sanity-check that the resulting positive rate is one the operation can absorb |
| `class_weight` | the per-class weights $w_k$ | `None`, meaning uniform | `None` leaves the objective unweighted; `"balanced"` sets $w_k = m/(K m_k)$; a dict sets each class by hand, and raising one class's entry raises that class's [[Recall]] and lowers its [[Precision]] | `"balanced"` as the first move when the target is imbalanced, a dict when the cost matrix says the ratio should be something other than the inverse prior |

Two settings pass the test and are pinned rather than tuned. `sample_weight` is the per-instance form of the same quantity and is set from the cost matrix or from the survey design that produced the rows, not searched. The `reduction` argument of a framework focal loss chooses between a mean, a sum and no aggregation, which changes the gradient scale and therefore interacts with the [[Learning Rate]]; pick `"mean"` and leave it.

## Failure modes

- **Reweighting and resampling applied together.** Oversample the minority to parity and then also pass `class_weight="balanced"`, and the correction is applied twice: the resampled counts already satisfy $m_k = m/K$, so the computed weights are near one, but a weight dict fixed from the *original* counts is not, and the minority ends up over-corrected by roughly the imbalance ratio. The scikit-learn case is worse because it is silent: `class_weight` and `sample_weight` are multiplied together where both are given, which `HistGradientBoostingClassifier` documents and most callers do not read.
- **Weights fitted to the training mix when the deployment mix differs.** $w_k = m/(K m_k)$ is computed from the training counts, so it corrects toward a uniform prior on the training data. If the deployment population has a different base rate, and it usually does when the training set was itself built by oversampling a rare event, the model is being corrected toward the wrong target. Elkan's Theorem 2 is the statement of how a predicted probability transforms under a change of base rate, and it is the thing to apply instead of assuming the training mix is the world's.
- **A cost matrix invented to make the numbers come out.** Because $p^{*} = C_{\text{FP}}/(C_{\text{FP}} + C_{\text{FN}})$ is a one-to-one map between the cost ratio and the threshold, any threshold anyone wants can be justified after the fact by quoting a cost ratio. A matrix assembled backwards from a desired operating point carries no information and launders a preference as an analysis. The test is whether each entry was elicited, in a stated unit against a stated baseline, from whoever pays it.
- **Focal loss with $\gamma$ high enough that mislabelled rows take over.** The modulating factor is a function of $p_t$ alone, and a correctly labelled hard instance and a mislabelled easy one both present as low $p_t$. At $\gamma = 2$ an instance at $p_t = 0.05$ is weighted $(0.95)^2 = 0.9$ against $0.01$ for one at $p_t = 0.9$, a ratio of $90$; at $\gamma = 5$ the same pair is weighted $0.774$ against $10^{-5}$, a ratio of about $77{,}000$. Past some $\gamma$ the gradient is essentially the label noise, and the symptom is a training loss that plateaus while the rare-class metrics decay.
- **Calibration quietly destroyed.** Reweighting changes the minimizer, so the fitted $\hat{p}$ is no longer an estimate of $P(y = 1 \mid \mathbf{x})$ under the training distribution, it is an estimate under the reweighted one. Any threshold chosen earlier is now on a different scale, and [[Log Loss]] read on the reweighted model is not comparable to [[Log Loss]] read on the unweighted one, because it is scoring a different quantity. Anything downstream that consumes the probability rather than the label, an expected-value calculation above all, is reading a number that has moved.
- **The effective sample size collapses.** Weights that vary a lot cost variance, and the amount is Kish's effective sample size, $m_{\text{eff}} = \left(\sum_i w_i\right)^2 / \sum_i w_i^2$, which equals $m$ only when the weights are all equal. On the [[MNIST]] 5-versus-rest target, $m = 60{,}000$ with $5{,}421$ fives gives $w_{5} = 5.534$ and $w_{\text{not }5} = 0.5497$, and

  $$m_{\text{eff}} = \frac{60{,}000^2}{5{,}421 \cdot 5.534^2 + 54{,}579 \cdot 0.5497^2} \approx 19{,}700$$

  so a tenfold imbalance buys its correction at the price of two thirds of the statistical weight of the data. At an imbalance ratio of $10^3$ the same arithmetic leaves a few thousand instances' worth of information in a set of millions, and the fitted model is correspondingly unstable across folds.

## Implementation

scikit-learn 1.6 covers the first two cases and not the third. Two arguments do the work and they sit in different places: `class_weight` is a constructor parameter set once per class, and `sample_weight` is a `fit` argument carrying one number per row. Where both are given they multiply.

Neither is universal. `class_weight` is available on `LogisticRegression`, `SGDClassifier`, `SVC` and `LinearSVC`, `Perceptron`, `RidgeClassifier`, the tree classifiers, `RandomForestClassifier` and `ExtraTreesClassifier`, and on `HistGradientBoostingClassifier` since 1.2. It is absent from `KNeighborsClassifier`, `MLPClassifier` and the naive Bayes estimators. `sample_weight` in `fit` is the wider of the two but still not everywhere: `KNeighborsClassifier.fit(X, y)` and `MLPClassifier.fit(X, y)` take no such argument in 1.6, and for those two the only route to a reweighted objective is [[Resampling]].

Computing the balanced weights explicitly, which is worth doing once to see the numbers rather than passing the string blind. scikit-learn 1.6:

```python
import numpy as np
from sklearn.utils.class_weight import compute_class_weight

classes = np.unique(y_train)
weights = compute_class_weight(class_weight="balanced", classes=classes, y=y_train)
dict(zip(classes, weights))     # {0: 0.5497, 1: 5.5340} on the MNIST 5-vs-rest target
```

The documented formula is `n_samples / (n_classes * np.bincount(y))`, which is $m / (K m_k)$ written in numpy. Passing `class_weight="balanced"` to an estimator runs exactly this and nothing else, here on [[Logistic Regression]], whose weighted objective is the [[Log Loss]] of the formula at the top of this note rather than its plain mean:

```python
from sklearn.linear_model import LogisticRegression

clf = LogisticRegression(class_weight="balanced").fit(X_train, y_train)
clf = LogisticRegression(class_weight={0: 1.0, 1: 12.0}).fit(X_train, y_train)
```

The cost-sensitive route, where the weight comes from the cost matrix rather than from the counts. With a zero diagonal, the weight on an instance is the cost of misclassifying it, which depends only on its true class. scikit-learn 1.6:

```python
import numpy as np
from sklearn.ensemble import HistGradientBoostingClassifier

c_fp, c_fn = 1.0, 12.0                                  # cost of a false alarm, cost of a miss
sample_weight = np.where(y_train == 1, c_fn, c_fp)

clf = HistGradientBoostingClassifier().fit(X_train, y_train, sample_weight=sample_weight)
```

Elkan's recommendation is the shorter alternative, and it touches the fit not at all. Train unweighted, then move the threshold to $p^{*}$:

```python
p_star = c_fp / (c_fp + c_fn)                           # 0.07692
proba = clf.predict_proba(X_test)[:, 1]
y_pred = proba >= p_star
```

`sample_weight` also propagates through `sklearn.metrics`, so a cost-weighted score is available without any change to the estimator, and the same array can be passed to `accuracy_score`, `log_loss` and the rest. Note that weighting the metric and weighting the fit are separate decisions that happen to share an argument name.

**scikit-learn 1.6 has no focal loss.** Nothing in `sklearn.metrics` implements it, and no estimator exposes it as a loss option: `SGDClassifier` offers `hinge`, `log_loss`, `modified_huber`, `squared_hinge` and `perceptron`, and `HistGradientBoostingClassifier` offers `log_loss` alone. This is not an oversight. Focal loss reweights per step from the model's current output, which needs a training loop that recomputes the loss each iteration, so it lives in the deep learning frameworks.

In PyTorch it is `torchvision.ops.sigmoid_focal_loss`, whose signature `sigmoid_focal_loss(inputs, targets, alpha=0.25, gamma=2, reduction='none')` is unchanged from torchvision 0.12 through the current stable release. The defaults are Lin et al.'s reported settings, `inputs` are raw logits rather than probabilities, and `reduction` defaults to `'none'`, so an unreduced tensor comes back and calling `.backward()` on it directly will fail:

```python
from torchvision.ops import sigmoid_focal_loss

loss = sigmoid_focal_loss(logits, targets.float(),
                          alpha=0.25, gamma=2.0, reduction="mean")
```

In Keras 3 it is a loss class rather than a function, `keras.losses.BinaryFocalCrossentropy(apply_class_balancing=False, alpha=0.25, gamma=2.0, from_logits=False, ...)` and `keras.losses.CategoricalFocalCrossentropy(alpha=0.25, gamma=2.0, from_logits=False, ...)`. The binary form keeps $\alpha$ switched off by default behind `apply_class_balancing=False`, so the plain $\gamma$-only loss is what you get unless you ask for the $\alpha$-balanced one, which is the opposite of Lin et al.'s own configuration and worth setting explicitly.
