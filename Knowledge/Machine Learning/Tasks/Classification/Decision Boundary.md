---
note_kind: concept
aliases:
  - decision boundary
  - decision boundaries
  - decision surface
  - decision region
  - decision regions
  - separating hyperplane
  - class boundary
up: "[[Classification]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The decision boundary is the set of points in feature space where a classifier's predicted class changes: the locus where the two best-scoring classes tie, so that an arbitrarily small move sends the prediction to a different label. It is not something a model outputs, it is something a fitted model *implies*, and it is the most useful picture of what a model family can and cannot express, because the shape it is allowed to take is exactly what separates one family from another.

## Formal statement

### The boundary is a tie between scores

Almost every classifier works by computing one score per class and predicting the largest, $\hat{y} = \arg\max_k s_k(\mathbf{x})$. The region assigned to class $k$ is where its score wins,

$$R_k = \big\{\, \mathbf{x} \in \mathcal{X} : s_k(\mathbf{x}) > s_j(\mathbf{x}) \ \ \text{for all } j \ne k \,\big\}$$

and the boundary is what those open regions leave over: the points where the strict inequality fails. Between a specific pair of classes it is the tie set

$$B_{jk} = \big\{\, \mathbf{x} : s_j(\mathbf{x}) = s_k(\mathbf{x}) \,\big\}$$

and the decision boundary of the whole model is the union of those pairwise tie sets, restricted to the parts where the tied pair is actually the winning pair. Two consequences follow immediately. The boundary depends only on *differences* of scores, so any transformation applied identically to all the scores and preserving their order leaves it untouched, which is why running scores through a softmax never moves it. And the boundary itself has no predicted class, being a tie; the library breaks ties by a fixed rule, and since the set has zero volume it never matters in practice.

### The binary case, and where the threshold lives

With two classes the whole thing reduces to one score against one cut point $t$, the two-stage form set out in [[Binary Classification]], and the boundary is the level set

$$B = \big\{\, \mathbf{x} : s(\mathbf{x}) = t \,\big\}$$

For a **linear** model $s(\mathbf{x}) = \boldsymbol\theta^{T}\mathbf{x}$ this is a hyperplane: a flat set of dimension $n - 1$ in an $n$-dimensional feature space, a point on a line, a line in a plane, a plane in a space. Its orientation is set by $\boldsymbol\theta$, which is the normal vector to it, and that is where the fitted knowledge lives.

[[Logistic Regression]] scores a probability rather than a raw number, and the boundary is still a hyperplane because the sigmoid is strictly increasing and therefore invertible:

$$\hat{p} = \sigma\big(\boldsymbol\theta^{T}\mathbf{x}\big) \ge t \iff \boldsymbol\theta^{T}\mathbf{x} \ge \log\frac{t}{1-t}$$

At the conventional $t = 0.5$ the right-hand side is $\log 1 = 0$, so the boundary is exactly $\boldsymbol\theta^{T}\mathbf{x} = 0$. This identity is the quantitative content of the whole note, and it settles what happens when the threshold is tuned:

$$\boldsymbol\theta^{T}\mathbf{x} = \log\frac{t}{1-t}$$

Moving $t$ changes the constant on the right and nothing else. The normal vector $\boldsymbol\theta$ is untouched, so **the hyperplane translates along its own normal and never rotates.** Improving the orientation requires refitting, or a different model; what the sweep can and cannot buy is set out in [[Precision-Recall Tradeoff]].

A [[Stochastic Gradient Descent Classifier]] cuts the same geometry at $t = 0$ on a raw score, and the note for that estimator carries what `decision_function` returns and how its sign relates to the sides of the plane.

The concrete instance: fitting logistic regression to [[Iris]] on petal width alone, one feature, so the hyperplane is a single point, and it falls at a petal width of $1.66$ cm. On the two petal measurements, the boundary is the line $\theta_0 + \theta_1 x_1 + \theta_2 x_2 = 0$, plotted by solving for the second coordinate as $x_2 = -(\theta_0 + \theta_1 x_1)/\theta_2$.

### Many classes: piecewise linear, and every region convex

For [[Softmax Regression]] each class carries its own linear score $s_k(\mathbf{x}) = (\boldsymbol\theta^{(k)})^{T}\mathbf{x}$, so each pairwise tie set is a hyperplane of its own:

$$B_{jk} = \big\{\, \mathbf{x} : \big(\boldsymbol\theta^{(j)} - \boldsymbol\theta^{(k)}\big)^{T}\mathbf{x} = 0 \,\big\}$$

There are $\binom{K}{2}$ of these, and the visible boundary is made of flat pieces cut from them, meeting along ridges where three or more scores tie at once. The regions are therefore bounded by straight faces rather than by one hyperplane, which is what "piecewise linear" means here.

Each region is also convex, and this is worth proving rather than asserting. Rewrite $R_k$ as an intersection:

$$R_k = \bigcap_{j \ne k} \big\{\, \mathbf{x} : \big(\boldsymbol\theta^{(k)} - \boldsymbol\theta^{(j)}\big)^{T}\mathbf{x} > 0 \,\big\}$$

Each set in that intersection is an open half-space, which is convex, and an intersection of convex sets is convex. So for any two points $\mathbf{u}, \mathbf{v} \in R_k$ the whole segment $\lambda\mathbf{u} + (1-\lambda)\mathbf{v}$, $\lambda \in [0,1]$, is in $R_k$ too. That is a hard limit on what this model family can represent: a class whose true extent is a ring, or two separated blobs, cannot be captured, because no convex set has either shape. It is also why a linear multiclass model can never leave an unreachable pocket of one class stranded inside another's territory.

### Curved boundaries from a model that is still linear

The linearity is in the *parameters*, not in the original features. Apply a feature map $\boldsymbol\phi$ and fit the same linear model on top of it; the boundary is a hyperplane in the transformed space,

$$\boldsymbol\theta^{T}\boldsymbol\phi(\mathbf{x}) = 0$$

and its preimage in the original feature space is whatever curve or surface that equation describes there. With the polynomial features of [[Polynomial Regression]] at degree $d$ the boundary is an algebraic surface of degree $d$: circles, ellipses and parabolas at $d = 2$, and progressively more contorted shapes as $d$ rises. Nothing about the fitting procedure changed, only the columns it was handed. This is the reason the linear families keep working well past the point where a straight line is plausible. Support vector machines and decision trees each reach curved boundaries by their own route, neither of which is a feature map, and neither has a note.

Boundary shape is therefore the honest summary of model capacity, and reading it is how [[Overfitting]] is diagnosed by eye: a boundary that detours around individual training points, or that grows thin fingers to capture a handful of instances, has fitted noise, and it is a fitted boundary, not a data property, so nothing in the data will show the same shape again.

### In scikit-learn

scikit-learn 1.6 plots the boundary of any fitted classifier directly, provided the input has exactly two features, since the routine evaluates the estimator on a grid over the plane:

```python
import matplotlib.pyplot as plt
from sklearn.inspection import DecisionBoundaryDisplay

disp = DecisionBoundaryDisplay.from_estimator(
    softmax_reg, X,
    response_method="predict",      # regions; "predict_proba" draws the contours instead
    xlabel="Petal length (cm)", ylabel="Petal width (cm)",
    alpha=0.5,
)
disp.ax_.scatter(X[:, 0], X[:, 1], c=y, edgecolor="k")
plt.show()
```

Two of its 1.6 defaults decide what is drawn rather than how it looks: `response_method='auto'` and `class_of_interest=None`. `response_method="predict"` fills the decision regions and is the one that draws the boundary as defined above; `"predict_proba"` or `"decision_function"` draws level sets of the underlying score instead, of which the boundary is one particular level. On a multiclass estimator with a score-based `response_method`, `class_of_interest` picks which class's score is drawn, since there are $K$ of them and only one surface can be plotted at a time.

## Where it is used

[[Classification]] is the parent task, and this is the geometric statement of what every classifier under it produces: [[Binary Classification]] gives one boundary separating two regions, and [[Multiclass Classification]] gives a partition into $K$ regions with a boundary between each adjacent pair.

[[Logistic Regression]] and the [[Stochastic Gradient Descent Classifier]] both draw a single hyperplane, differing only in what the score means and where the cut sits, and [[Softmax Regression]] draws $\binom{K}{2}$ of them into a piecewise linear partition whose regions are convex. [[Precision-Recall Tradeoff]] is the same object seen from the metrics side: sweeping the threshold slides the hyperplane along its normal, which is why that sweep can rebalance the two kinds of error and can never improve the fit. [[Polynomial Regression]] is the cheapest route to a curved boundary, and [[Overfitting]] is what a boundary looks like when it has learned the noise.
