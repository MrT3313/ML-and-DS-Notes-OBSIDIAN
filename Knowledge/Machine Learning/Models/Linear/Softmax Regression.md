---
note_kind: method
aliases:
  - softmax
  - softmax regression
  - multinomial logistic regression
  - multinomial logit
  - softmax classifier
  - maximum entropy classifier
  - maxent classifier
up: "[[Multiclass Classification]]"
sources:
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Softmax regression is [[Logistic Regression]] with the two-class restriction lifted. Instead of one linear score for one positive class, it fits one linear score per class, converts the whole vector of scores into a probability distribution, and predicts the class with the largest of them. Everything else about the model is unchanged: the scores are linear in the features, the loss is [[Log Loss]], and there is no closed form.

Reach for it on a [[Multiclass Classification]] target when you want one fitted model rather than a stack of binary ones. It replaces [[One-versus-Rest]]'s $K$ classifiers and [[One-versus-One]]'s $\frac{K(K-1)}{2}$ classifiers with a single parameter matrix trained in a single optimization, and the probabilities it returns sum to $1$ across the classes by construction rather than being $K$ independently fitted numbers renormalized after the fact. A decomposition can produce a set of binary answers that disagree with each other, three classifiers each claiming their own class; softmax regression cannot, because the classes compete inside one objective.

Its restriction is the other half of the same fact: one class per instance. It is multiclass, not multioutput, so it is the wrong model for [[Multilabel Classification]].

## Algorithm or formula

Each class $k$ gets its own parameter vector, and the score for that class is the ordinary linear score built from it:

$$s_k(\mathbf{x}) = \big(\boldsymbol\theta^{(k)}\big)^{T} \mathbf{x}, \qquad k = 1, \dots, K$$

Stack those $K$ vectors as the rows of a **parameter matrix** $\mathbf{\Theta}$, of shape $K \times (n + 1)$ with $n$ features plus the bias term, and the whole score vector for one instance is one matrix-vector product:

$$\mathbf{s}(\mathbf{x}) = \mathbf{\Theta}\,\mathbf{x} \in \mathbb{R}^{K}$$

so a design matrix $\mathbf{X}$ of $m$ rows yields all $m \times K$ scores as $\mathbf{X}\mathbf{\Theta}^{T}$. That parameter matrix is the whole of what is fitted, and the count is worth noticing: $K(n+1)$ numbers, against the $n+1$ of a binary model, so the parameter count grows linearly in the number of classes rather than the model count growing.

Turning $\mathbf{s}(\mathbf{x})$ into probabilities is the softmax function, which exponentiates every score and divides by their total so the results are positive and sum to one:

$$\hat{p}_k = \sigma\big(\mathbf{s}(\mathbf{x})\big)_k = \frac{e^{s_k(\mathbf{x})}}{\sum_{j=1}^{K} e^{s_j(\mathbf{x})}}$$

$K$ is the number of classes, $\mathbf{s}(\mathbf{x})$ the vector of all $K$ scores for the instance $\mathbf{x}$, and $\sigma(\mathbf{s}(\mathbf{x}))_k$ the estimated probability that $\mathbf{x}$ belongs to class $k$ given those scores. [[Multiclass Classification]] carries the general treatment of turning a score vector into a distribution; what is specific here is that the scores being fed in are linear.

Predicting is the $\arg\max$ over the probabilities, and the consequence worth drawing out is that this makes them decoratively rather than operationally important: softmax is strictly increasing in $s_k$, so

$$\hat{y} = \arg\max_{k} \hat{p}_k = \arg\max_{k} s_k(\mathbf{x})$$

The prediction could be read straight off the raw scores. The probabilities exist because you want to know how confident the model is, and because the loss needs them.

### What is fitted

The objective is the $K$-class cross-entropy form of [[Log Loss]], minimized over $\mathbf{\Theta}$; the formula lives in that note. The gradient with respect to one class's parameter vector is the part that belongs here, because it is a statement about this parameterization:

$$\nabla_{\boldsymbol\theta^{(k)}} J(\mathbf{\Theta}) = \frac{1}{m} \sum_{i=1}^{m} \Big( \hat{p}_k^{(i)} - y_k^{(i)} \Big) \mathbf{x}^{(i)}$$

One gradient vector per class, each of them the training instances averaged with weights equal to that class's signed probability error. The structure is identical to the linear regression gradient with the residual $\hat{y} - y$ replaced by the probability residual $\hat{p}_k - y_k$, and it says the obvious thing: an instance pulls on class $k$'s parameters in proportion to how much probability that class wrongly claimed or wrongly withheld. Stacking the $K$ rows gives $\nabla_{\mathbf{\Theta}} J = \frac{1}{m} (\hat{\mathbf{P}} - \mathbf{Y})^{T} \mathbf{X}$ with $\mathbf{Y}$ the one-hot target matrix, which is the form the code actually computes.

There is no closed form for $\mathbf{\Theta}$, so the fit is [[Gradient Descent]] or a second-order solver, and the [[Convexity]] of the loss is what makes either of them worth running.

### The $K = 2$ case is logistic regression

Softmax reads only *differences* of scores, so adding the same vector $\mathbf{c}$ to every class's parameters leaves every $\hat{p}_k$ unchanged and $\mathbf{\Theta}$ is identified only up to that shift. Take $K = 2$ and shift by $\mathbf{c} = -\boldsymbol\theta^{(1)}$:

$$\hat{p}_2 = \frac{e^{(\boldsymbol\theta^{(2)} - \boldsymbol\theta^{(1)})^{T}\mathbf{x}}}{1 + e^{(\boldsymbol\theta^{(2)} - \boldsymbol\theta^{(1)})^{T}\mathbf{x}}} = \sigma\big(\boldsymbol\theta^{T}\mathbf{x}\big), \qquad \boldsymbol\theta = \boldsymbol\theta^{(2)} - \boldsymbol\theta^{(1)}$$

which is exactly [[Logistic Regression]]. Not an analogy: the same model, written with one redundant parameter vector. ($\sigma$ there is the logistic sigmoid $\sigma(t) = 1/(1 + e^{-t})$. The softmax is never written $\sigma$ in these notes, since the same symbol elsewhere is a standard deviation; [[Notation]] carries the split.)

The redundancy has a practical consequence. With no penalty the minimizer is not unique, since the whole shift direction is flat, and the weights can drift along it without the loss changing. Any [[Regularization]] term breaks the tie by preferring the smallest-norm representative, which is one reason scikit-learn penalizes by default rather than offering an unpenalized fit as the baseline.

### The boundaries it draws

Setting two class scores equal gives $\big(\boldsymbol\theta^{(j)} - \boldsymbol\theta^{(k)}\big)^{T}\mathbf{x} = 0$, a hyperplane, so the regions this model assigns are bounded by flat pieces and each region is convex. [[Decision Boundary]] carries that argument.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| inverse regularization strength | $C$ | `1.0` | less penalty on $\mathbf{\Theta}$, larger weights, sharper probabilities and a more flexible set of boundaries, more [[Overfitting]] risk. Large $C$ also restores the identifiability problem above, since the tie-breaking term shrinks | [[Cross-Validation]] over a log grid, ex `np.logspace(-4, 4, 9)` |
| solver | `solver` | `"lbfgs"` | not ordered, and it changes the fitted model rather than only the speed: `"liblinear"` cannot fit the multinomial loss and falls back to one-versus-rest, so it fits a different model from the other five | leave at `"lbfgs"`; `"saga"` for `"elasticnet"` or very large $m$; `"newton-cholesky"` when $m \gg nK$, which is the condition the user guide states, though its memory grows as $(nK)^{2}$ because it forms the full Hessian |
| class weights | `class_weight` | `None` | `"balanced"` weights class $k$ by $m / (K\, m_k)$, so rare classes contribute more per instance and their recall rises at the cost of the common classes' precision | set it under [[Class Imbalance]], then read a [[Confusion Matrix]] rather than accuracy |
| maximum iterations | `max_iter` | `100` | more solver steps, longer fit, more chance of actually converging. The default is low enough that unscaled data routinely hits it | raise it until the convergence warning stops, or scale the features instead |

## Failure modes

- **Unscaled features stall the fit at `max_iter=100`.** One step size serves every coordinate, so a feature in the thousands and a feature in the unit interval cannot both be stepped well, and the $\ell_2$ penalty charges the same per unit of weight regardless of the column's unit, which silently penalizes small-scale features harder. The symptom is a `ConvergenceWarning` and a model whose coefficients change when you raise `max_iter`. Put a [[Standardization]] step in front of it in a [[Pipeline]].
- **Nearly separable classes plus a large $C$ send weights toward infinity.** No finite minimizer exists on separable data, for the reason [[Logistic Regression]] sets out; only the penalty stops it, so `C=1e9` and `penalty=None` are the settings that produce it. The visible symptom here is that the probabilities harden toward $0$ and $1$ and the boundary is placed by whichever instances happen to sit nearest it.
- **Reading the probabilities as calibrated.** Softmax output sums to $1$ and looks like a posterior, but nothing enforces that $\hat{p}_k$ matches the empirical frequency of class $k$ among instances scored $\hat{p}_k$. Regularization systematically shrinks the scores and therefore flattens the probabilities, and the $\arg\max$ is unaffected either way, so an over-regularized model can look accurate and be badly calibrated. [[Model Calibration]] is where that gap stops being a warning and becomes a number: it supplies the reliability diagram and the expected calibration error that measure it, and the maps fitted on held-out scores that close it. Take the measurement before any of these numbers reach a decision rule.
- **Using it where the labels are not mutually exclusive.** A multilabel target passed to `LogisticRegression` does not raise a helpful error about the modelling assumption, it just fits something whose output space cannot represent the answer. The check is on the target, not the model: `type_of_target(y) == "multilabel-indicator"` means the wrong estimator.
- **Assuming the strategy is fixed across versions.** The `multi_class` argument was deprecated in scikit-learn 1.5 and is scheduled for removal in 1.7, and `newton-cholesky` gained multinomial support only in 1.6. Code that was correct on 1.4 can fit a different model on 1.6 without any warning being read, so the version matters as much as the arguments.

## Implementation

scikit-learn 1.6: there is no softmax estimator and no argument to switch on. `LogisticRegression` fitted on a target with three or more classes already minimizes the full multinomial loss under every solver except `liblinear`, which only does one-versus-rest. The two petal measurements of [[Iris]] are the standing example, with three classes:

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

iris = load_iris(as_frame=True)
X = iris.data[["petal length (cm)", "petal width (cm)"]].values
y = iris["target"]
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

softmax_reg = LogisticRegression(C=30, random_state=42)
softmax_reg.fit(X_train, y_train)

softmax_reg.predict([[5, 2]])                    # array([2])
softmax_reg.predict_proba([[5, 2]]).round(2)     # array([[0. , 0.04, 0.96]])
```

`coef_` comes back with shape `(K, n)` and `intercept_` with shape `(K,)`, which is the parameter matrix $\mathbf{\Theta}$ split into its weight block and its bias column. `C=30` is a deliberately weak penalty, chosen to let the boundaries follow the data closely on a set this small; it is not a default worth carrying elsewhere.

scikit-learn 1.6, the version-proof spelling and the one to write now, since the `multi_class` argument disappears in 1.7 and passing it already warns:

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.multiclass import OneVsRestClassifier

# Multinomial softmax, stated by doing nothing, with scaling in front.
softmax_reg = make_pipeline(StandardScaler(),
                            LogisticRegression(C=30, random_state=42))

# One-versus-rest instead, which from 1.7 is the only way to ask for it.
ovr_reg = OneVsRestClassifier(LogisticRegression(random_state=42))
```
