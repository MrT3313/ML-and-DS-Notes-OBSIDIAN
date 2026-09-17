---
note_kind: meta
title: "Notation"
---

# Notation

- _lower case italic_ $\rightarrow$ scalar values & function names
- **lower case bold** $\rightarrow$ vectors
- **UPPER CASE BOLD** $\rightarrow$ matricies
- a bold lower case vector is a _column vector_ by default $\rightarrow$ a 2D array with a single column, shape $n \times 1$, unless it is explicitly transposed

| term / symbol                        | desc                                                                                                                                                                                                  |
|--------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| $m$                                  | number of instances                                                                                                                                                                                   |
| $n$ | number of features $\rightarrow$ an instance vector $\mathbf{x}$ carries $n$ feature values, and $\boldsymbol\theta$ carries $n+1$ entries once the bias term is counted |
| $\textbf{x}^i$                       | vector of all feature values (excluding the label) of the $i^{th}$ instance in the dataset                                                                                                            |
| $y^i$                                | label / desired output of the $x^i$ instance in the dataset                                                                                                                                           |
| $\mathbf{X}$                         | a matrix containing all the features values (excluding labels) of all instances in the dataset. 1 row per $i^{th}$ instance $=$ transpose of $\mathbf{x}^{(i)}$ $\rightarrow$ $(\mathbf{x}^{(i)})^T$  |
| $h$                                  | prediction function / _hypothesis_ $\rightarrow$ when given instance vector $\mathbf{x}^{(1)}$ its predicted output is $\hat{y}^{(i)} = h(\mathbf{x}^{(i)})$                                          |
| $\mathbf{u}, \mathbf{v}, \mathbf{w}$ | arbitrary vectors in a purely mathematical statement, no machine learning role $\rightarrow$ distinct from $\mathbf{x}^{i}$, which is always an instance's feature vector                             |
| $\boldsymbol\theta$                  | the model _parameter vector_, holding the bias term $\theta_0$ and the feature weights $\theta_1$ to $\theta_n$, as a column of shape $(n+1) \times 1$                                                |
| $\boldsymbol\theta^{T}$              | the _transpose_ of $\boldsymbol\theta$ $\rightarrow$ the same numbers laid out as a single row, shape $1 \times (n+1)$                                                                                |
| $\boldsymbol\theta^{T}\mathbf{x}$    | the **linear score** $\rightarrow$ the matrix multiplication of a $1 \times (n+1)$ row by an $(n+1) \times 1$ column, giving a $1 \times 1$ result read as the scalar $\theta_0 x_0 + \theta_1 x_1 + \dots + \theta_n x_n$. It is the quantity every linear model computes first, and it is the prediction $\hat{y}$ only where the link is the identity, which is the [[Linear Regression]] case. A classifier hands it on to a sigmoid, a softmax or a threshold, so there it is a score and not yet an answer |
| $\boldsymbol\theta \cdot \mathbf{x}$ | the dot product of the two vectors, the _same number_ as $\boldsymbol\theta^{T}\mathbf{x}$ above. Both spellings appear in the vault: the dot product form says what the quantity is, the transpose form says how the arrays are shaped when it is computed |
| $\partial$ | the partial derivative sign $\rightarrow$ $\partial f / \partial \theta_j$ is the rate of change of $f$ when $\theta_j$ alone moves and every other variable is held fixed. The rounded $\partial$ marks it as partial, against the straight $d$ of a single-variable derivative. [[Partial Derivative]] carries it |
| $\nabla$ | the gradient operator, read _nabla_ or _del_ $\rightarrow$ $\nabla_{\boldsymbol\theta} f$ stacks one partial derivative per parameter into a column of $n+1$ entries, each row taking its own index: $\partial f/\partial\theta_0$ at the top through $\partial f/\partial\theta_n$ at the bottom, never one repeated $j$. It is not the capital $\Delta$, which denotes a finite difference or a change in a quantity and is a different operator; writing $\Delta_{\boldsymbol\theta}$ for a gradient is a symbol error rather than a variant spelling. [[Gradient]] carries it |
| $\epsilon$ | the [[Tolerance]] $\rightarrow$ the threshold below which an improvement or a gradient norm counts as small enough to stop on, scikit-learn's `tol`. Distinct from $\varepsilon$, which this vault reserves for the irreducible noise term of a data-generating process |
| $\hat{p}$ | an estimated probability $\rightarrow$ what a classifier reports for a class, against $\hat{y}$ for a predicted value or label. $\hat{p}_k$ is the probability given to class $k$ |
| $K$ | number of classes in a [[Multiclass Classification]] target, against $k$ for one particular class |
| $\hat{\boldsymbol\theta}$ | the _fitted_ parameter vector $\rightarrow$ the particular $\boldsymbol\theta$ that minimizes the cost on the training data, as against $\boldsymbol\theta$ standing for the parameters in general |
| $\eta$ | the [[Learning Rate]] $\rightarrow$ the scalar an update multiplies the gradient by before subtracting it. $\eta_0$ is the initial rate, scikit-learn's `eta0`, and $\eta_t$ the rate on update $t$ under a [[Learning Schedule]] |
| $\alpha$ | the [[Regularization]] strength $\rightarrow$ the weight on the penalty term of a regularized cost, and scikit-learn's `alpha` argument. Much of the statistics literature writes the same quantity $\lambda$, and `LogisticRegression` and `SVC` expose its inverse $C$ instead; this vault uses $\alpha$ throughout, leaving $\lambda$ to its other standard jobs, eigenvalues and convex-combination weights among them |
| $\sigma$ | three conventional meanings, separated by what the symbol is applied to. On a scalar score it is the logistic sigmoid $\sigma(t) = 1/(1 + e^{-t})$. Subscripted by a feature, as $\sigma_j$, or attached to a distribution, as $\operatorname{Var}(\varepsilon) = \sigma^{2}$, it is a standard deviation. Some sources also write the vector-to-vector softmax as $\sigma$; this vault does not, and spells softmax out as its ratio of exponentials or names it in words |
| $t$ | three readings, all standard, separated by context. Under a [[Learning Schedule]] it is the update counter, equal to $1$ on the first update. Applied to a score it is the decision threshold that turns $s(\mathbf{x})$ into a label. As the argument of the sigmoid, $\sigma(t)$, it is the linear score being squashed |
