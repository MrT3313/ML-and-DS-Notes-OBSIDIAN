---
note_kind: concept
aliases:
  - scaling
  - feature normalization
  - feature scaling
  - scale features
  - rescaling
  - scaler
up: "[[Feature Engineering]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Feature scaling maps each numeric column onto a comparable range so that no [[Feature]] outweighs the others through its unit of measurement alone. Whether it matters at all is a property of the algorithm rather than of the data: anything that measures distance, follows a gradient, or charges one shared penalty across weights is sensitive to scale, and anything that tests one feature against a threshold at a time is not.

## Formal statement

Every scaler in this family is one affine map per column, with the constants estimated once and then frozen:

$$x'_j = \frac{x_j - a_j}{b_j}$$

Min-max scaling takes $a_j = \min_j$ and $b_j = \max_j - \min_j$; standardization takes $a_j = \mu_j$ and $b_j = \sigma_j$. Three mechanisms make $b_j$ matter.

**Distance.** $\lVert \mathbf{x}^{(i)} - \mathbf{x}^{(k)} \rVert_2^2 = \sum_j \big(x^{(i)}_j - x^{(k)}_j\big)^2$ sums squared differences across columns (see [[Lp Norm]]), so a column measured in tens of thousands contributes terms of order $10^8$ while a column in $[0,1]$ contributes terms of order $1$, and the second column is effectively ignored. k-nearest neighbors, k-means and the RBF kernel of a support vector machine inherit this directly.

**Curvature.** For a squared-error [[Cost Function]] the Hessian is proportional to $\mathbf{X}^{T}\mathbf{X}$. Columns of very different scale give it eigenvalues of very different size, the contours stretch into a ravine, and [[Gradient Descent]] zigzags across the narrow direction instead of running down it. The largest eigenvalue caps the usable [[Learning Rate]], so the slowest direction sets the step count.

**Penalty.** A [[Regularization]] term such as $\alpha \sum_j \theta_j^2$ charges every weight at one rate, which is only fair if a unit of each feature means a comparable amount. [[Ridge Regression]] and [[Lasso Regression]] are the two standing cases, and neither is safe to fit on raw columns.

Decision trees and tree ensembles use none of the three. A split tests $x_j \le t$, and any strictly increasing rescale carries $t$ along with it, so the tree is unchanged.

The sharpest illustration that this is a property of the solver rather than of the model is [[Linear Regression]], which can be fitted two ways. Gradient descent on it wants scaled columns; the [[Normal Equation]] and the SVD route want nothing at all. Write a rescale as $\mathbf{X}' = \mathbf{X}\mathbf{D}$ with $\mathbf{D}$ diagonal and positive. Then $\boldsymbol\theta' = (\mathbf{X}'^{T}\mathbf{X}')^{-1}\mathbf{X}'^{T}\mathbf{y} = \mathbf{D}^{-1}\boldsymbol\theta$, so the coefficients change but the fitted predictions $\mathbf{X}'\boldsymbol\theta' = \mathbf{X}\boldsymbol\theta$ do not. Same model, same data, one solver that cares and one that does not. Two things keep the claim honest. The invariance is algebraic and is not free in floating point, for a reason [[Normal Equation]] sets out in the conditioning of $\mathbf{X}^{T}\mathbf{X}$. And the invariance dies the moment a penalty is added, since $\alpha \sum_j \theta_j^2$ is written in the units of $\boldsymbol\theta$ and $\mathbf{D}^{-1}\boldsymbol\theta$ is not charged the same as $\boldsymbol\theta$.

## Where it is used

Scaling belongs inside a [[Pipeline]], which refits the scaler on whatever data it is handed and replays the stored constants on the rest, making the discipline automatic rather than remembered. Fit on the [[Training Set]] only: computing $\min$, $\max$, $\mu$ or $\sigma$ over the full dataset lets statistics of the [[Testing Set]] reach the model, which is [[Data Leakage]] and is one of its six named causes. What is contaminated there is the fitted transformer rather than any judgement you made, so it happens with nobody looking at anything. The constants are also a description of the training distribution and nothing more, so a serving distribution that has moved away from it leaves them describing a population that is no longer there: scaled values start landing outside the range the model was fitted on, and no adjustment to the scaler repairs that, only a refit on newer data. This is [[Model Rot]] arriving at the preprocessing step rather than at the weights, and it is part of why a deployed model carries a retraining cadence at all, which is the standing practice [[Continual Learning]] describes. [[Min-Max Scaling]] is the bounded option, [[Standardization]] the unbounded one. Neither changes the shape of a distribution, so a [[Skewed Data|skewed]] column is still skewed afterwards and wants [[Feature Distribution Transformation]] first. [[Missing Value Imputation]] also comes first, since the constants cannot be computed across missing entries. Labels get the same treatment through [[Target Scaling]], and [[Ensemble Learning]] over trees is where you can skip all of it.
