---
note_kind: method
aliases:
  - stacking
  - stacked generalization
  - stacked regression
  - stacked ensemble
  - metalearner
  - meta-model
  - blending
  - StackingClassifier
  - StackingRegressor
  - out-of-fold
  - out-of-fold predictions
up: "[[Ensemble Learning]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Stacking fits base learners on the training data and then fits a further model, the metalearner, whose inputs are the base learners' outputs and whose output is the final prediction. The metalearner can be as simple as a heuristic, a fixed average or a hand-set weighting, or it can be another model with its own fit. What makes the family worth having is the model case: instead of deciding in advance how much each member's opinion is worth, you learn it, and you can learn it as a function of what the members said rather than as a single constant per member.

That is the one thing [[Ensemble Learning]] cannot get from averaging or voting. Both of those fix the combination in advance and spend the diversity of the members uniformly; stacking spends it selectively, learning that one member is reliable where another is not. It is also the family that most rewards genuinely different member types, since the metalearner has nothing to learn from members that agree, which is the $\rho$ in the variance formula appearing as a design instruction.

Reach for it when several dissimilar candidates are all competitive and you would otherwise pick one by [[Model Selection]]. Breiman's observation on his own experiments ("Stacked Regressions", *Machine Learning* 24(1), 1996) is that stacking never did worse than selecting the single best predictor and that the biggest gains came when dissimilar sets of predictors were stacked. Do not reach for it when one candidate dominates the rest, when the members are variants of one algorithm, or when the deployment budget cannot carry $M+1$ models plus the [[Cross-Validation]] pass needed to fit the last one honestly.

## Algorithm

Write the training split as $D$ with rows indexed $i = 1, \dots, m$, the base learners as $h_1, \dots, h_M$, and the metalearner as $g$.

1. **Choose the folds.** Partition $D$ into $K$ folds $F_1, \dots, F_K$, and write $k(i)$ for the fold containing row $i$.
2. **Fit each base learner $K$ times, once per fold, each time excluding that fold.** Write $h_j^{-k}$ for base learner $j$ fitted on $D \setminus F_k$.
3. **Build the metalearner's design matrix from the held-out predictions.** Row $i$ of $Z$ is
   $$\mathbf{z}^{(i)} = \Big(h_1^{-k(i)}\big(\mathbf{x}^{(i)}\big), \; \dots, \; h_M^{-k(i)}\big(\mathbf{x}^{(i)}\big)\Big)$$
   so every entry is a prediction made by a fit that excluded the row it is predicting. These are the out-of-fold predictions, and the condition they satisfy is the whole content of the note.
4. **Fit the metalearner** $g$ on $(Z, \mathbf{y})$, with the original targets unchanged.
5. **Refit each base learner on all of $D$.** These full-data fits are what will answer queries; the $K$ fold fits existed only to build $Z$ and are discarded.
6. **Predict** by passing a new $\mathbf{x}$ through the full-data base learners and their outputs through $g$:
   $$\hat{y}(\mathbf{x}) = g\Big(h_1(\mathbf{x}), \dots, h_M(\mathbf{x})\Big)$$

Steps 3 and 6 use base learners fitted on different amounts of data, and the asymmetry is deliberate rather than an oversight: the metalearner has to be taught on predictions that behave like predictions about unseen rows, because that is what it will be handed at prediction time, while the predictions it is actually handed should come from the best fits available. That asymmetry is in the method from the beginning, in Wolpert's construction ("Stacked Generalization", *Neural Networks* 5(2), 1992), which builds the level-1 learning set from the level-0 generalizers taught on one part of a partition and questioned on the other, and answers a query with the same generalizers taught on the whole learning set.

### The condition, stated so it can be falsified

Everything above reduces to one requirement on $Z$, and it is the only thing in the note that a wrong answer cannot be recovered from.

> [!important]
> **For every row $i$ and every base learner $j$, the fit that produced the entry $z^{(i)}_j$ must not have been trained on row $i$.**

Written out: $z^{(i)}_j = h_j^{-k}(\mathbf{x}^{(i)})$ for some $k$ with $i \in F_k$, and therefore $(\mathbf{x}^{(i)}, y^{(i)}) \notin \text{train}\big(h_j^{-k}\big)$. This is checkable rather than a matter of care. Name a row and a base learner, look up which fit produced that cell, and ask whether the row was in that fit's training set; a single cell where it was falsifies the condition for the whole matrix, and no averaging over the other cells repairs it.

What goes wrong when it fails is specific. A base learner with the capacity to overfit reproduces the training labels of the rows it was fitted on, so for such a member $z^{(i)}_j \approx y^{(i)}$ on exactly the rows the metalearner is being trained on. That column is then a near-copy of the target, dressed as a feature. The metalearner does what it is supposed to do with such a column and puts almost all of its weight on it, and the weights it learns describe a relationship that holds on the training rows and nowhere else. At prediction time that member is predicting a row it has not seen, its output is an ordinary imperfect prediction, and the combination has been calibrated for an input distribution that no longer exists.

That is [[Data Leakage]] arriving inside an ensemble, and it is an instance of the general condition rather than an analogy: the axiom that a target is never legitimate for itself, $y \notin \text{legit}\{y\}$, is broken by a feature column computed by a procedure that was shown $y$ for the row it is describing. It is also the harder kind to notice, because nothing in the shape of the data reveals it. The design matrix has the right dimensions, the fit converges, and the cross-validated score of the *stacked* model can look excellent if the same contaminated predictions are used to compute it.

Breiman is the source of the requirement stated as a requirement. His level-1 data is built by leaving out row $n$, refitting every predictor without it, and recording what those refits say about it, $z_{kn} = v_k^{(-n)}(\mathbf{x}_n)$, and his reason for doing it that way is stated directly: if the predictors are built on the full learning set and the combination coefficients are then chosen by minimizing squared error on that same set, the coefficients "will overfit the data" and generalization will be poor. Modern practice replaces his leave-one-out construction with $K$-fold, and that is not merely a cost saving: he tested both and reports that the 10-fold level-1 data gave more accurate performance than the leave-one-out version.

Which source carries which claim is worth keeping straight, because the attribution is routinely compressed. Wolpert makes the held-out requirement *definitional*, since the two halves of each of his partitions are disjoint by construction, but he states no prohibition against building the level-1 data from in-sample predictions, and he is candid that much of the rest of the scheme is unsettled, with no rules for which generalizers to use at either level. The requirement stated as a requirement, with a reason attached, is Breiman's. The explicit modern warning, in so many words, is scikit-learn's, and it is quoted under the failure modes below. Wolpert's typeset journal pages are paywalled, so the wording attributed to him here was checked against the Los Alamos preprint of the same paper rather than against the printed version.

### Combining is not an unconstrained regression

The second half of Breiman's paper is a finding that is easy to lose and expensive to rediscover: even with honest out-of-fold predictions, fitting the combination by ordinary least squares does not reliably work. The reason is that the columns of $Z$ are strongly correlated by construction, because every base learner is trying to predict the same thing, so the coefficients are unstable against small changes in the data.

His answer is not a penalty. He reports trying ridge regression and variants of it, getting results better than plain least squares but not consistent, and settles instead on least squares under a non-negativity constraint,

$$\min_{\boldsymbol\alpha} \sum_{i=1}^{m}\Big(y^{(i)} - \sum_{j=1}^{M}\alpha_j z^{(i)}_j\Big)^{2} \quad \text{subject to} \quad \alpha_j \ge 0 \ \ \text{for all } j$$

and reports that this "performed by far the best" of everything he tried. The mechanism he gives for why is geometric rather than statistical. Add the further constraint $\sum_j \alpha_j = 1$ and the combination becomes an interpolating predictor, bounded at every $\mathbf{x}$ by

$$\min_j v_j(\mathbf{x}) \;\le\; \sum_{j} \alpha_j v_j(\mathbf{x}) \;\le\; \max_j v_j(\mathbf{x})$$

so the combination can never answer outside the range of what its members answered, which is exactly the failure an unconstrained fit on correlated columns produces. He also shows the sum-to-one constraint is unnecessary in practice, since the non-negative optimum comes out with $\sum_j \alpha_j \approx 1$ on its own, and notes that the constraint makes the solution sparse: across his simulations the number of non-zero coefficients averaged around three.

One exact result is worth carrying, because it says when stacking cannot help. Writing $R_{ik}$ for the relevant cross-moments of the predictors' errors, the best single predictor $v_k$ is also the best stacked predictor if and only if $R_{kk} \le R_{ik}$ for every $i$, which in words means every predictor with error comparable to $v_k$ has to be nearly equal to $v_k$. That is the formal version of "stacking pays when the members disagree", and it is the same condition the correlation floor in [[Ensemble Learning]] expresses.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| the metalearner, `final_estimator` | $g$ | `LogisticRegression` on `StackingClassifier`, `RidgeCV` on `StackingRegressor` | not a scale. Raising the metalearner's capacity lets it learn a combination that varies with what the members said rather than one weight per member, and it also lets it overfit $Z$, which has only $m$ rows and few columns. A high-capacity metalearner on a narrow $Z$ is the standard way to lose the gain | keep it simple and let the base learners carry the capacity. A penalized linear model is the default for a reason, and Breiman's finding is that the constraint that matters is non-negativity of the weights, which neither default imposes |
| the fold scheme, `cv` | $K$ | `None`, which means 5-fold, stratified when the estimator is a classifier and the target is binary or multiclass | more folds means each fold fit sees more data, so the out-of-fold predictions resemble the full-data predictions more closely and the metalearner is taught on a distribution nearer the one it will meet. Cost is $M \times K$ base fits plus $M$ final refits | 5 is the default and usually enough. Raise it when $m$ is small, since a fold fit on four fifths of a small file is visibly weaker than the full-data fit that replaces it. Do not lower it below 3, and never set `"prefit"` to save time |
| the members' response method, `stack_method` | | `'auto'`, which tries `predict_proba`, then `decision_function`, then `predict` | not a scale. It decides what the columns of $Z$ actually contain. `predict_proba` gives the metalearner a continuous score per class and so more to work with than a hard label; `predict` collapses each member to its decision and throws away its confidence | leave on `'auto'`, which already prefers the most informative method each member offers. Pin it explicitly only when the members must be made comparable, and note it raises if a member does not implement the method named |
| original features alongside, `passthrough` | | `False` | `True` appends the original feature matrix to $Z$, so the metalearner can learn a combination that depends on where in feature space the query sits, ex trusting one member only in a region. It widens $Z$ from $M$ columns to $M + p$, which is how a simple metalearner starts overfitting | leave `False` unless there is a reason to think member reliability varies by region, and score the change on held-out folds rather than assuming. With a wide feature matrix this turns the metalearner into a model fitted on $p$ features plus a few hints |

`random_state` does not appear on these estimators. It belongs to the members and to the splitter passed as `cv`: an unseeded shuffled splitter changes which rows land out of fold, which changes every entry of $Z$ and therefore the fitted metalearner. Pin it on the splitter, see [[Random Seed]]. Note that an integer or `None` `cv` instantiates its splitter with `shuffle=False`, so the folds are contiguous blocks of the row order and repeat identically across calls, which is reproducible and is wrong for a file sorted by class or by time.

## Failure modes

- **Fitting the metalearner on in-fold predictions.** The condition above, broken. Fit the base learners on all of $D$, ask them to predict $D$, and hand those columns to the metalearner, and a member that can memorize has just handed it the answer key; the metalearner weights that member almost exclusively, and at prediction time that weight is attached to an ordinary prediction. scikit-learn exposes this as `cv="prefit"` and documents the hazard in its own words, that if the models were trained on the same data used to train the stacking model there is a very high risk of overfitting. The argument exists for the case where the members were fitted on genuinely separate data, and using it otherwise is the failure mode with a keyword.
- **Scoring the stacked model on the same predictions that trained it.** A second, subtler version of the same error. Even with honest out-of-fold columns, a score computed from $Z$ by the metalearner that was fitted on $Z$ is a training score. The stacked estimator has to go through [[Cross-Validation]] as a single unit, with the fold structure outside it, so that the whole procedure including the metalearner's fit is repeated per fold. In scikit-learn that means passing the `StackingClassifier` itself to `cross_val_score`, which produces $M \times K$ inner fits per outer fold and is the real cost of the method.
- **An unconstrained metalearner on correlated columns.** The members all predict the same target, so the columns of $Z$ are strongly correlated and ordinary least squares on them produces large coefficients of opposing signs that are unstable against small data changes. The symptom is a combination that can answer outside the range of every member's answer, and the fix is a constraint or a penalty rather than more data.
- **Members that are variants of one algorithm.** Three gradient-boosted tree models with different seeds give the metalearner three nearly identical columns, and by Breiman's condition above there is nothing to gain from combining predictors that are nearly equal wherever their errors are comparable. The method costs $M \times K + M + 1$ fits and returns approximately the single best member.
- **Leakage entering through the fold structure rather than through the predictions.** Everything above assumes the folds are honest. With time-correlated rows or grouped rows, the out-of-fold prediction for row $i$ comes from a fit that contained row $i$'s sibling or row $i$'s future, so the condition holds as written and fails in substance. Use the splitter the data demands, `GroupKFold` or a time-ordered split, and pass it as `cv` rather than an integer.
- **Preprocessing fitted outside the fold loop.** A scaler or an imputer fitted on all of $D$ before the base learners are stacked leaks the held-out fold's statistics into every fold fit, so the out-of-fold predictions are not out of fold in the sense that matters. Each base learner has to be a complete [[Pipeline]], which is the same requirement cross-validation carries and is easier to forget here, because there are $M$ of them.
- **Deployment cost counted as one model.** A stacked ensemble is $M$ base models plus the metalearner, all of which must be loaded, versioned and called for every prediction, and the fit cost is $M \times K + M$ model fits rather than $M$. This is the case [[Ensemble Learning]] makes about operational cost against a small gain, at its most expensive point in the family.

## Implementation

scikit-learn 1.6:

```python
from sklearn.ensemble import StackingClassifier, RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.model_selection import StratifiedKFold, cross_val_score

base = [
    ("forest", RandomForestClassifier(n_estimators=200, random_state=42)),
    ("svc", SVC(probability=True, random_state=42)),
    ("logreg", LogisticRegression(max_iter=1000)),
]

# the splitter is where the seed goes: it decides which rows land out of fold
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

stack = StackingClassifier(
    estimators=base,
    final_estimator=LogisticRegression(max_iter=1000),
    cv=cv,                  # out-of-fold predictions come from here
    stack_method="auto",    # predict_proba where a member offers it
    passthrough=False,
)

# score the whole thing as one unit, folds outside the stack
cross_val_score(stack, X_train, y_train, cv=cv, scoring="accuracy")
```

The 1.6 signatures are `StackingClassifier(estimators, final_estimator=None, *, cv=None, stack_method='auto', n_jobs=None, passthrough=False, verbose=0)` and `StackingRegressor(estimators, final_estimator=None, *, cv=None, n_jobs=None, passthrough=False, verbose=0)`. The regressor has no `stack_method` because a regressor has only `predict`. `final_estimator=None` means `LogisticRegression` on the classifier and `RidgeCV` on the regressor, so the regressor's default metalearner is a cross-validated penalized linear fit on out-of-fold predictions, which is Breiman's recipe with a ridge penalty in place of his non-negativity constraint. Neither default constrains the weights to be non-negative, so that part of his finding is something to impose yourself if you want it.

The sentence in the documentation that carries the condition of this note is worth knowing verbatim, because it is the library stating which fits produce what: "Note that `estimators_` are fitted on the full `X` while `final_estimator_` is trained using cross-validated predictions of the base estimators using `cross_val_predict`." That is steps 3 and 5 above, and it means the out-of-fold requirement is enforced by the estimator rather than left to the caller, as long as `cv` is not set to `"prefit"`.

`cv` accepts `None` for 5-fold, an integer, a splitter object, an iterable of index pairs, or the string `"prefit"`, added in 1.1. Under an integer or `None` the splitter is `StratifiedKFold` when the estimator is a classifier and the target is binary or multiclass, and `KFold` otherwise, instantiated without shuffling. `"prefit"` assumes every member is already fitted and trains the metalearner on their predictions over the full training set, which is the leak named above; the documentation says so in the same paragraph.

`passthrough=True` appends the original `X` to the base learners' predictions before the metalearner sees it. `stack_method` takes `'auto'`, `'predict_proba'`, `'decision_function'` or `'predict'`, and a named method that a member does not implement raises rather than falling back.

The word **blending** is used for the cheaper variant in which a single held-out split replaces the $K$ folds: the base learners are fitted on one part, predict the other, and the metalearner is fitted on that one set of held-out predictions. It satisfies the condition of this note and costs $M + M$ fits instead of $M \times K + M$, paying for it with a metalearner fitted on a fraction of the rows and base learners fitted on a fraction of the data. There is no scikit-learn estimator for it; it is `train_test_split` plus two fits, written by hand.
