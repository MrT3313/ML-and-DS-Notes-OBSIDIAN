---
note_kind: concept
aliases:
  - log loss
  - logloss
  - cross-entropy
  - cross entropy
  - cross-entropy loss
  - binary cross-entropy
  - categorical cross-entropy
  - log_loss
  - neg_log_loss
  - logistic loss
  - negative log-likelihood
up: "[[Cost Function]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

Log loss scores a predicted *probability*, not a predicted label. For one instance it is the negative logarithm of the probability the model assigned to the class that actually occurred, so a confident correct answer costs almost nothing and a confident wrong answer costs without bound. Averaged over a set it is the [[Cost Function]] that [[Logistic Regression]] and [[Softmax Regression]] are fitted by, and it is also a reportable metric. It is the same number in both roles, which is unusual: most models are trained on one quantity and judged on another.

## Formal statement

### Two classes

Let $\hat{p} = P(y = 1 \mid \mathbf{x})$ be the model's estimate for the positive class. Per instance the loss is two cases:

$$c(\boldsymbol\theta) = \begin{cases} -\log(\hat{p}) & \text{if } y = 1 \\[2pt] -\log(1 - \hat{p}) & \text{if } y = 0 \end{cases}$$

Both branches say one thing: find the probability the model gave to the outcome that happened, and take minus its log. Since the model reports $\hat{p}$ for class $1$, the probability it implicitly gave to class $0$ is $1 - \hat{p}$, which is why the second branch is not $-\log(\hat{p})$ with a sign flipped.

Because $y \in \{0, 1\}$, the two branches collapse into one expression, each factor switching off the branch that does not apply:

$$\ell(\hat{p}, y) = -\Big[\, y \log \hat{p} + (1 - y) \log(1 - \hat{p}) \,\Big]$$

Set $y = 1$ and the second term vanishes; set $y = 0$ and the first does. Averaging that per-instance loss over the training set, in the shape every [[Cost Function]] takes, gives log loss proper:

$$J(\boldsymbol\theta) = -\frac{1}{m} \sum_{i=1}^{m} \Big[\, y^{(i)} \log \hat{p}^{(i)} + \big(1 - y^{(i)}\big) \log\big(1 - \hat{p}^{(i)}\big) \,\Big]$$

### $K$ classes

With $K$ classes the target is one-hot, $y_k^{(i)} = 1$ when instance $i$ belongs to class $k$ and $0$ otherwise, and the model reports a full distribution $\hat{p}_1^{(i)}, \dots, \hat{p}_K^{(i)}$. The loss generalizes by summing over classes:

$$J(\mathbf{\Theta}) = -\frac{1}{m} \sum_{i=1}^{m} \sum_{k=1}^{K} y_k^{(i)} \log \hat{p}_k^{(i)}$$

The inner sum has exactly one surviving term per instance, the one where $y_k^{(i)} = 1$, so this too is "minus the log of the probability given to the truth", averaged. Setting $K = 2$ and writing the two one-hot coordinates as $y$ and $1 - y$ returns the binary expression above, so there are not two losses here, only one written at two levels of generality. This is the form [[Softmax Regression]] minimizes.

### Why the logarithm, and in what units

$-\log$ is strictly decreasing on $(0, 1]$ with

$$-\log(1) = 0, \qquad \lim_{\hat{p} \to 0^{+}} -\log(\hat{p}) = +\infty$$

so the only way to score zero is to put all the mass on the right answer, and asserting near-certainty about something false is unboundedly expensive. That asymmetry is deliberate: no bounded loss can punish a confidently wrong probability hard enough to stop a model from bluffing. The logarithm is natural (base $e$), so the units are nats. Dividing by $\log 2$ converts to bits, which is the form information theory usually writes.

### Why a probability metric rather than accuracy

Log loss is a **strictly proper scoring rule**: it is minimized, uniquely, by reporting the true probabilities, so shading them in either direction costs something. That is the whole reason to fit and rank probability estimates with this rather than with [[Accuracy]]. Accuracy reads only $\arg\max_k p_k$, so a model that says $0.51$ and a model that says $0.99$ for the same correct class score identically, and a model can improve its probabilities without accuracy moving at all.

### Convexity in the parameters

In $\hat{p}$ alone the loss is convex by inspection, $-\log$ being convex on $(0,1]$. What matters for fitting is convexity in $\boldsymbol\theta$, which is a property of the composition $\hat{p} = \sigma(\boldsymbol\theta^{T}\mathbf{x})$ rather than of the loss on its own, and therefore belongs to the model: [[Logistic Regression]] carries the Hessian and what it rules out. The consequence to know here is that minimizing this loss over a linear score has no spurious local minima, so [[Gradient Descent]] suffices and the absence of a closed form costs nothing.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import log_loss
from sklearn.model_selection import cross_val_score

log_loss(y_test, clf.predict_proba(X_test))

losses = -cross_val_score(clf, X_train, y_train,
                          scoring="neg_log_loss", cv=5)
```

The 1.6 signature is `log_loss(y_true, y_pred, *, normalize=True, sample_weight=None, labels=None)`. `y_pred` is an array of probabilities of shape `(n_samples, n_classes)`, straight from `predict_proba`, or a 1D array read as the positive class's probability in the binary case. The column order is the classes sorted, matching `LabelBinarizer`, so passing `predict_proba` output from a fitted estimator lines up and hand-built arrays are where the ordering goes wrong.

`eps` is gone. It was deprecated in 1.3 and removed in 1.5, so 1.6 has no such argument: predictions are clipped internally to $[\varepsilon, 1 - \varepsilon]$ with $\varepsilon$ the machine precision of the array's dtype, which is what keeps a reported probability of exactly $0$ on the true class from returning infinity. Rows of `y_pred` that do not sum to $1$ are not rejected: the function warns, `The y_pred values do not sum to one. Make sure to pass probabilities.`, and returns a number anyway, so a scorer wired to an uncalibrated score rather than to `predict_proba` reports a plausible-looking loss that means nothing.

`labels` names the full class list. It is needed whenever `y_true` does not contain every class, which happens on small folds and on rare classes, because without it the classes are inferred from `y_true` alone and the probability columns are then matched against the wrong labels. A `y_true` with only one distinct label raises unless `labels` is supplied.

Every scorer follows the convention that higher is better, so the scorer string is `neg_log_loss` and the returned values are negated losses. The scikit-learn 1.6 model evaluation guide lists log loss among the strictly consistent scoring functions for classification. Consistency is defined relative to a functional and propriety relative to the whole predicted distribution, so the two notions are not interchangeable in general; for log loss they coincide, and the practical reading is the one stated above.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `normalize` | none | `True` | `True` returns the mean loss per instance; `False` returns the sum, so the value scales with $m$ and stops being comparable across sets of different size | leave `True` for anything you compare across sets |
| `labels` | the class list $\{1, \dots, K\}$ | `None` | naming more classes than appear in `y_true` changes which probability column is matched to which label, so a wrong list silently scores the wrong column | pass `clf.classes_` whenever a fold or a rare class might not contain every label |
| `sample_weight` | per-instance weights $w_i$ | `None` | the average becomes $\sum_i w_i \ell_i / \sum_i w_i$, so heavily weighted instances dominate the score | leave unset unless the instances genuinely are not equally important |

## Where it is used

It is the [[Cost Function]] that [[Logistic Regression]] minimizes in its binary form and that [[Softmax Regression]] minimizes in its $K$-class form, and in both cases the minimizing is done by [[Gradient Descent]] because neither has a closed form. [[Convexity]] is the property that makes that minimization worth attempting at all, and the Hessian above is where that property comes from rather than being asserted.

[[Accuracy]] is the contrast that explains why it exists: accuracy scores only the decision, log loss scores the probability behind the decision, and only the second is a strictly proper scoring rule. As a [[Performance Measure]] it is therefore the right thing to rank candidate models by when the probabilities themselves are going to be used, ex when a threshold will be set later or when the output feeds a cost calculation. [[Cross-Validation]] is how it is read honestly, through `scoring="neg_log_loss"`, which keeps the estimate off the data the model was fitted on.
