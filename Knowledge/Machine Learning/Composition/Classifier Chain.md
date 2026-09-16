---
note_kind: method
aliases:
  - classifier chain
  - classifier chains
  - chain of classifiers
  - ClassifierChain
  - sklearn.multioutput.ClassifierChain
  - CC
up: "[[Multilabel Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

A classifier chain gives [[Multilabel Classification]] to a classifier that has none. It arranges $L$ binary classifiers in a fixed order and feeds each one the original features plus the labels of the models before it in the chain, so model $j$ is allowed to condition its answer on what models $1$ through $j-1$ said. It also serves [[Multioutput Classification]], the generalization where each of the $L$ targets is multiclass rather than binary, since nothing in the construction requires the passed-forward labels to be binary.

Reach for it when the base classifier cannot take a label matrix. `SVC` is the standing case: in scikit-learn 1.6 the estimators with native multilabel support are the trees and forests, `KNeighborsClassifier`, `RadiusNeighborsClassifier`, `MLPClassifier` and `RidgeClassifier`, and `SVC` is not among them, so it has to be wrapped.

The choice is against `MultiOutputClassifier`, which is the simpler wrapper for the same gap: it fits one independent classifier per label and stacks their answers. That is cheaper and parallel, and it assumes the labels are independent given the features, which for the pair of MNIST labels used below, "the digit is 7 or larger" and "the digit is odd", is plainly false. A chain targets the difference. Writing $\pi$ for the chain order, it decomposes the joint by the chain rule,

$$P(y_1, \dots, y_L \mid \mathbf{x}) = \prod_{j=1}^{L} P\big(y_{\pi(j)} \mid \mathbf{x},\, y_{\pi(1)}, \dots, y_{\pi(j-1)}\big)$$

where the independent-per-label wrapper fits $\prod_j P(y_j \mid \mathbf{x})$ instead. Everything a chain can gain over it lives in the conditioning terms, and everything it can lose lives there too.

It is an estimator wrapping an estimator, so the [[Scikit-Learn Estimator API]] holds in both directions, the same contract [[Pipeline]] relies on: the chain clones the base estimator $L$ times, exposes one `fit` and one `predict` returning an $m \times L$ matrix, and the base estimator's own settings are reachable through `base_estimator__param`.

## Algorithm

Let $\mathbf{X} \in \mathbb{R}^{m \times d}$ and let $\mathbf{Y} \in \{0,1\}^{m \times L}$ be the label matrix.

1. Fix the order $\pi$, a permutation of $0, \dots, L-1$. The default is the column order of $\mathbf{Y}$, so $\pi = [0, 1, \dots, L-1]$; an explicit list or `"random"` replaces it.
2. Clone the base estimator $L$ times, once per link.
3. Fit link $j = 1$ on $\mathbf{X}$ alone, with target column $\pi(1)$. It has no predecessors, so it is an ordinary binary fit.
4. Fit link $j$ on the augmented design matrix
$$\mathbf{X}^{(j)} = \big[\, \mathbf{X} \;\big|\; \mathbf{c}_{\pi(1)}, \dots, \mathbf{c}_{\pi(j-1)} \,\big] \in \mathbb{R}^{m \times (d + j - 1)}$$
  with target column $\pi(j)$. The appended columns $\mathbf{c}$ are what makes this a chain rather than $L$ independent fits, and where they come from is the `cv` decision of step 5.
5. Choose what fills those columns. With `cv=None`, $\mathbf{c}_{\pi(t)}$ is the **true** label column $\mathbf{Y}[:, \pi(t)]$. With `cv` set to $k$ folds, $\mathbf{c}_{\pi(t)}$ is instead the out-of-fold output of `cross_val_predict` for link $t$, so the column holds predictions made by models that did not see the row they are predicting.
6. Repeat 4 and 5 for $j = 2, \dots, L$, each link inheriting one more column than the last.
7. To predict, run the chain in the same order: link 1 predicts from $\mathbf{x}$, its output is appended, link 2 predicts from the augmented row, and so on until all $L$ columns exist. Reorder the columns back to the original label order before returning.

> [!warning]
> Steps 5 and 7 are where the two `cv` settings genuinely differ. With `cv=None` the chain is trained on true predecessor labels but must predict from *predicted* ones, and those are not the same distribution: at training time column $\mathbf{c}_{\pi(1)}$ is perfect, at prediction time it carries link 1's error rate. Later links learn to trust a feature that will be less reliable than it looked, which is a train and predict mismatch, and it flatters cross-validated scores. Setting `cv` is what removes it, because the columns then carry realistic errors during training too. It is the reason to pass `cv=3` rather than leave the default.

What `cv` costs. The chain performs $L$ final fits, plus a $k$-fold [[Cross-Validation]] run for each of the first $L-1$ links, so the total number of base-estimator fits is

$$L + k\,(L - 1)$$

against $L$ for `cv=None`. At $L = 2$ with `cv=3` that is $2 + 3 = 5$ fits of `SVC`.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| wrapped base estimator | `base_estimator` | required, positional, no default | a higher-capacity base classifier changes every one of the $L$ fitted links and multiplies through the $L + k(L-1)$ fits | [[Cross-Validation]] over candidates, with each candidate's settings reached as `base_estimator__C` |
| chain order | `order` | `None`, meaning the column order $[0, 1, \dots, L-1]$ | a different permutation changes which labels are available to which link, so it changes every fit after the first; `"random"` draws one of the $L!$ orderings | try several orders and average, or ensemble independently ordered chains; there is no order that is right a priori |
| cross-validation for the passed-forward columns | `cv` | `None`, meaning the true predecessor labels | `None` removes the cost and keeps the mismatch; an integer $k$ replaces true labels with out-of-fold predictions, and a larger $k$ makes each fold's model closer to the final fit, at $k(L-1)$ extra base fits | 3 or 5 is enough to remove the mismatch; raise it only if the per-fold training sets are too small to be representative |
| response method passed forward | `chain_method` | `"predict"` | `"predict"` appends hard 0/1 labels; `"predict_proba"`, `"predict_log_proba"` or `"decision_function"` append a continuous score, which carries how *confident* the earlier link was rather than only what it decided, and a list of names falls through to the first one the base estimator implements | leave at `"predict"` unless the base estimator produces well-behaved scores, then compare under cross-validation |

`verbose` only prints progress and is not a hyperparameter.

## Failure modes

- **Leaving `cv` at its default trains the chain on information it will not have.** With `cv=None` every link after the first is fitted against error-free predecessor columns and then asked, at prediction time, to read columns produced by fallible models. A link can put large weight on a predecessor label that is only 90% accurate in the field and be rewarded for it during fitting. The symptom is a model that scores better in development than in use, and the fix is to pass `cv`.
- **Errors propagate forward and never get corrected.** Link $j$'s mistake becomes an input feature for links $j+1, \dots, L$, and no later link can revise it, because the chain is a single forward pass. Under the crude assumption that each link is correct with probability $p$ independently, the chance that a whole chain of $L$ links is clean falls as $p^{L}$, so long chains degrade even with individually strong links. `MultiOutputClassifier` has no such path, which is the honest counterpoint to the dependence modelling.
- **The result depends on an order nobody chose deliberately.** The default is whatever order the label columns happen to sit in, one of $L!$ possibilities, and swapping two labels can change the predictions. A chain that looks good may be reporting a lucky permutation, which is why the usual remedy is an ensemble of chains with different orders rather than a single chain.
- **The base estimator's cost is paid $L + k(L-1)$ times, on a design matrix that grows.** `SVC` fit time "scales at least quadratically with the number of samples", so a chain over `SVC` is restricted to the first 2,000 rows for the same reason a [[One-versus-Rest]] wrapper around `SVC` is: the full 60,000-row MNIST training set would be impractical. Nothing about the chain reduces that; it multiplies it.
- **A label that is nearly constant poisons its successors.** If one label is positive in a tiny fraction of rows, its link predicts the majority class almost everywhere, and with `chain_method="predict"` the column it appends is a near-constant feature. Later links gain a column with no signal and still pay for it in the fit.

## Implementation

scikit-learn 1.6:

```python
from sklearn.multioutput import ClassifierChain
from sklearn.svm import SVC

chain_clf = ClassifierChain(SVC(), cv=3, random_state=42)
chain_clf.fit(X_train[:2000], y_multilabel[:2000])

chain_clf.predict([some_digit])   # shape (1, 2), one column per label
```

The full signature in 1.6 is `ClassifierChain(base_estimator, *, order=None, cv=None, chain_method="predict", random_state=None, verbose=False)`.

`random_state` is reproducibility rather than configuration, so it is not in the table above, but it does two jobs here and both are worth knowing: it draws the permutation when `order="random"`, and it is passed down to each cloned base estimator that exposes a `random_state` of its own. With `order=None` the first job is idle. See [[Random Seed]] for why an integer rather than the `None` default is what makes a run repeatable.

`y_multilabel` here is an $m \times 2$ boolean matrix built from the MNIST digit labels, column 0 being "the digit is 7 or larger" and column 1 being "the digit is odd":

```python
import numpy as np

y_train_large = (y_train >= '7')
y_train_odd = (y_train.astype('int8') % 2 == 1)
y_multilabel = np.c_[y_train_large, y_train_odd]
```

Both columns are functions of the same underlying digit, which is exactly the label dependence a chain is built to exploit.
