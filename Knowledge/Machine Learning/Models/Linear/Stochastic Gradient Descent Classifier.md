---
note_kind: method
aliases:
  - SGD classifier
  - SGDClassifier
  - SGDC
  - SGD
  - stochastic gradient descent classifier
  - linear classifier fitted by SGD
  - sgd_clf
up: "[[Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

`SGDClassifier` is not a model family of its own. It is a linear classifier fitted by stochastic gradient descent, and the `loss` argument decides which linear model you are actually fitting: the default `hinge` gives a linear support vector machine, `log_loss` gives [[Logistic Regression]], `perceptron` gives the perceptron. The thing held constant across those choices is the optimizer, not the model.

Reach for it when the training set is large enough that a solver touching every instance per step is too slow, or when the data arrives as a stream. Updating on one instance at a time is what makes it an [[Online Learning]] estimator: it exposes `partial_fit`, and the scikit-learn guide to scaling lists it among the classifiers that train out of core, because no step ever needs more than the current batch in memory.

It performs [[Binary Classification]] natively, scoring one instance against one hyperplane. A multiclass target is handled by wrapping it in [[One-versus-Rest]], one binary classifier per class, and predicting whichever classifier reports the highest score.

## Algorithm or formula

The model is one linear score per instance,

$$s(\mathbf{x}) = \boldsymbol\theta^{T}\mathbf{x} + b$$

and the decision is that score against a threshold $t$:

$$\hat{y} = \begin{cases} 1 & \text{if } s(\mathbf{x}) > t \\ 0 & \text{otherwise} \end{cases}$$

The fixed default is $t = 0$, not $0.5$. The $0.5$ that appears in [[Logistic Regression]] is a threshold on a *probability*; `decision_function` returns a signed distance to the hyperplane, which runs over all of $\mathbb{R}$, and the sign of that number is exactly the side of the boundary the instance falls on. So $0$ is the natural cut and $0.5$ would be an arbitrary one.

What gets minimized is the regularized training error

$$E(\boldsymbol\theta, b) = \frac{1}{m}\sum_{i=1}^{m} L\big(y^{(i)}, s(\mathbf{x}^{(i)})\big) + \alpha R(\boldsymbol\theta)$$

with $L$ chosen by `loss` and $R$ by `penalty`. Ordinary gradient descent would evaluate that whole sum before moving. Stochastic gradient descent does not: it takes one instance $i$ (or one small batch) at a time and steps on that instance's gradient alone,

$$\boldsymbol\theta \leftarrow \boldsymbol\theta - \eta \left[ \alpha \nabla_{\boldsymbol\theta} R(\boldsymbol\theta) + \nabla_{\boldsymbol\theta} L\big(y^{(i)}, s(\mathbf{x}^{(i)})\big) \right]$$

so the cost of one update does not grow with $m$. The [[Learning Rate]] $\eta$ is not constant by default. Under `learning_rate="optimal"` it decays with the number of updates $s$ taken so far,

$$\eta_s = \frac{1}{\alpha \, (s + s_0)}$$

with $s_0$ set by a heuristic. Note that $\alpha$ therefore does two jobs at once under the default schedule: it is the regularization strength and it sets the step size.

### The threshold is exposed, not settable

scikit-learn does not let you set the threshold on the classifier directly. `predict()` always applies $t = 0$. What it does give you is `decision_function()`, which returns the raw scores, and from those you can threshold wherever you like.

scikit-learn 1.6:

```python
y_scores = sgd_clf.decision_function([some_digit])   # array([2164.22030239])
y_pred = (y_scores > 3000)                           # array([False])
```

That is the whole mechanism behind the [[Precision-Recall Tradeoff]]: sweep $t$ across the scores and read off what each setting costs. Running `cross_val_predict(..., method="decision_function")` gets an out-of-fold score for every training instance, which is what the sweep is computed on.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| loss function | `loss` | `"hinge"` | not ordered. `"hinge"` fits a linear SVM, `"log_loss"` fits [[Logistic Regression]] and unlocks `predict_proba`, `"modified_huber"` is smooth, outlier tolerant and also gives probabilities, `"perceptron"` fits a perceptron | pick by what you need out of the model, probabilities or a margin, then [[Cross-Validation]] between the survivors |
| regularization penalty | `penalty` | `"l2"` | not ordered. `"l1"` and `"elasticnet"` drive weights to exactly zero and so select features, `None` removes the penalty entirely | `"l2"` unless you want sparsity |
| regularization strength | $\alpha$ | `0.0001` | stronger penalty, smaller weights, more [[Underfitting]]. Under the default schedule it also shrinks the step size $\eta_s = 1/(\alpha(s+s_0))$, so raising it slows learning as well | log grid, the user guide suggests `10.0 ** -np.arange(1, 7)` |
| elastic net mixing | `l1_ratio` | `0.15` | more of the penalty is $\ell_1$, so a sparser weight vector. Only read when `penalty="elasticnet"` | log grid jointly with $\alpha$ |
| maximum epochs | `max_iter` | `1000` | more passes over the data, longer fit, more chance of converging before the cap | the guide's heuristic is `np.ceil(1e6 / m)`, since SGD tends to converge after roughly $10^{6}$ instances seen |
| stopping tolerance | `tol` | `0.001` | stops sooner on a smaller improvement, so an earlier and possibly underfit stop | lower it if the fit stops while still improving |
| learning rate schedule | `learning_rate` | `"optimal"` | not ordered. `"constant"` holds $\eta = \eta_0$, `"invscaling"` decays as $\eta_0 / s^{\,p}$, `"adaptive"` holds $\eta_0$ and divides it by 5 whenever progress stalls | leave at `"optimal"` unless the loss diverges or plateaus early |
| initial learning rate | $\eta_0$ | `0.0` | bigger steps, faster movement, risk of overshooting and diverging. Ignored by the default `"optimal"` schedule, which is why the default is $0$ | only set it once you have left `"optimal"`, then a log grid |
| inverse scaling exponent | `power_t` | `0.5` | faster decay of $\eta$, so less movement late in the fit. Only read when `learning_rate="invscaling"` | rarely worth touching |
| class weights | `class_weight` | `None` | `"balanced"` reweights each class by $m / (K \, m_k)$, so the rare class counts for more per instance and recall on it rises | set it when the target shows [[Class Imbalance]], then check [[Precision]] and [[Recall]], not accuracy |
| early stopping | `early_stopping` | `False` | `True` holds out a stratified slice and stops on the validation score instead of the training loss, which guards [[Overfitting]] at the cost of training data | turn on for large $m$ where the held-out slice is cheap |
| held-out fraction | `validation_fraction` | `0.1` | larger validation slice, less noisy stopping signal, less data to fit on. Only read when `early_stopping=True` | leave at 0.1 unless $m$ is small |
| patience | `n_iter_no_change` | `5` | waits longer through flat stretches before stopping, so a longer fit and less risk of stopping on noise | raise it when the loss curve is jumpy |
| iterate averaging | `average` | `False` | `True` stores the running average of the weights instead of the last iterate, which cuts the variance SGD leaves behind. An integer starts averaging after that many instances | try `True` when successive fits disagree with each other |
| fit intercept | `fit_intercept` | `True` | `False` forces $b = 0$ and the boundary through the origin, which is only right if the data is already centred | leave `True` |
| shuffle | `shuffle` | `True` | `False` keeps the instance order fixed across epochs, which lets any ordering in the file become a pattern in the fit | leave `True` |

`random_state` is left out of the table on purpose: it changes the result without being a quantity you tune. The shuffling and, under `early_stopping`, the validation split both draw on it, so two unseeded fits on identical data give different weights. Pin it with an integer. See [[Random Seed]].

## Failure modes

- **No probabilities under the default loss.** `predict_proba` is implemented only for `loss="log_loss"` and `loss="modified_huber"` and raises otherwise, so the out-of-the-box `hinge` fit gives you `decision_function` and nothing else. Anything downstream that wants a calibrated probability needs either a switch of loss or a `CalibratedClassifierCV` wrapper.
- **Unscaled features wreck the fit.** One learning rate $\eta$ multiplies the gradient for every coordinate, so a feature measured in thousands drags the step around while a feature in the unit interval barely registers, and the penalty $R(\boldsymbol\theta)$ charges the same price per unit of weight no matter what unit the column is in. The numbers on MNIST: the multiclass SGD classifier scores $[0.87365, 0.85835, 0.8689]$ under 3-fold [[Cross-Validation]] on raw pixels, and $[0.8983, 0.891, 0.9018]$ after [[Standardization]], about three points of accuracy for one line of [[Feature Scaling]].
- **High accuracy on an imbalanced target means nothing.** On the 5-versus-rest MNIST target it cross-validates to $[0.95035, 0.96035, 0.9604]$, which sounds strong until you notice a constant "not a 5" [[Baseline Model]] scores $0.90965$ on the same split. See [[Class Imbalance]].
- **Run-to-run drift without a seed.** Shuffling is on by default, so the instance order, and therefore the weights you end up with, change every fit unless `random_state` is pinned. A difference between two candidates that is smaller than that drift is not a difference.
- **Stopping early on a flat stretch.** `tol=0.001` with `n_iter_no_change=5` stops as soon as five epochs fail to improve the loss by that much, which on a slow-improving problem ends the fit well short of `max_iter`. Check `n_iter_` against `max_iter` before believing the model converged.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import SGDClassifier

sgd_clf = SGDClassifier(random_state=42)
sgd_clf.fit(X_train, y_train_5)

sgd_clf.predict([some_digit])          # array([True])
sgd_clf.decision_function([some_digit])  # array([2164.22030239])
```

`random_state=42` is the only argument set above, and it is there for reproducibility rather than for fit quality. See [[Random Seed]].

scikit-learn 1.6, always scale first in real use:

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

sgd_clf = make_pipeline(StandardScaler(), SGDClassifier(random_state=42))
```

scikit-learn 1.6, the current answer to "you cannot set the threshold directly". Version 1.5 added two estimators that make the threshold a fitted or declared property of a model rather than something you apply by hand to a score array. [[HOML Ch03 Classification|HOML chapter 3]] predates them and does the thresholding by hand.

```python
from sklearn.model_selection import FixedThresholdClassifier, TunedThresholdClassifierCV

# A threshold you have already chosen, wrapped so predict() honours it.
fixed = FixedThresholdClassifier(SGDClassifier(random_state=42),
                                 threshold=3000.0,
                                 response_method="decision_function")

# Or let cross-validation pick the cut that maximizes a metric you name.
tuned = TunedThresholdClassifierCV(SGDClassifier(random_state=42),
                                   scoring="f1", cv=3)
tuned.fit(X_train, y_train_5)
tuned.best_threshold_
```

`FixedThresholdClassifier` defaults to `threshold="auto"`, which resolves to $0$ when the response is `decision_function` and $0.5$ when it is `predict_proba`, matching the rule above. `TunedThresholdClassifierCV` defaults to `scoring="balanced_accuracy"` and searches 100 candidate thresholds by cross-validation.
