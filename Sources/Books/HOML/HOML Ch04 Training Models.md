---
note_kind: source
medium: book
chapter: 4
up: "[[HOML]]"
url: https://github.com/ageron/handson-ml3/blob/main/04_training_linear_models.ipynb
aliases:
  - HOML Chapter 4
  - Training Models
---

## Thesis

Every model in chapters 1 to 3 was fitted by calling `fit()` and taking the result on trust; chapter 4 is where that box is opened. It starts with the smallest case that has anything to open, a linear model under squared error, and shows that two quite different routes arrive at the same parameters: a closed form that solves in one matrix operation for the point where the gradient vanishes, and an iterative descent that starts the parameters anywhere and walks downhill until they stop moving. Neither route produces a better answer than the other, so nothing about the quality of the fit chooses between them. What chooses is the shape of the data, the instance count $m$ against the feature count $n$, together with whether all the rows can be held in memory at once, and that is also why the family of descents splits three ways over a single question, how many instances one gradient is computed on. The second half turns the same machinery on capacity instead of on cost. Adding powers and products of the existing columns lets a model that is still linear in its parameters trace a curve, which makes the degree a dial that can be turned too far; two error curves plotted together are how you see that it was turned too far, and where the excess actually lives; and regularization is the dial that pulls it back, charged either as a penalty on the weights inside the training objective or as a rule that stops the optimizer before it has finished. The chapter closes by removing the oddity of a classifier sitting in a chapter about regression. Put a squashing function on the linear score and replace squared error with log loss, and the same machine returns a probability rather than a quantity, for two classes or for $K$ of them, with what it predicts reducing in every case to a boundary drawn through feature space.

## Extracted

Fitting a linear model in closed form:
- [[Normal Equation]]

Fitting it by descent:
- [[Gradient Descent]]
- [[Batch Gradient Descent]]
- [[Stochastic Gradient Descent]]
- [[Mini-Batch Gradient Descent]]
- [[Learning Schedule]]
- [[Epoch]]
- [[Tolerance]]
- [[Parameter Space]]

Capacity and how it is read:
- [[Polynomial Regression]]
- [[Learning Curve]]
- [[Bias-Variance Tradeoff]]

Holding capacity back:
- [[Regularization]]
- [[Ridge Regression]]
- [[Lasso Regression]]
- [[Elastic Net Regression]]
- [[Early Stopping]]

The same linear machine as a classifier:
- [[Softmax Regression]]
- [[Log Loss]]
- [[Decision Boundary]]

Mathematics:
- [[Partial Derivative]]
- [[Gradient]]
- [[Convexity]]

The data:
- [[Iris]]

Amended, not produced, by this chapter:
- [[Linear Regression]]
- [[Logistic Regression]]
- [[Stochastic Gradient Descent Classifier]]
- [[Learning Rate]]
- [[Cost Function]]
- [[Overfitting]]
- [[Underfitting]]
- [[Generalization]]
- [[Model Selection]]
- [[Hyperparameter]]
- [[Feature Scaling]]
- [[Lp Norm]]
- [[Out-of-Core Learning]]
- [[Online Learning]]
- [[Multiclass Classification]]

Hubs and indexes revised to match what the chapter delivered, rather than notes it produced:
- [[Machine Learning]], the domain hub, whose `Optimization/` paragraph carried the forward promise this chapter is paying off
- [[Mathematics]], the mathematics hub
- [[Notation]], the symbol conventions, which gained the ones this chapter's formulas use
- [[Regression]], the task index, five new methods on its list
- [[Classification]], the task index, one new method on its list

Corrected by this chapter:
- [[Insufficient Training Data]]: it promised that a precise bound relating training set size to the generalization gap was chapter 4 material. Chapter 4 gives no such bound. That result belongs to statistical learning theory and [[HOML]] does not cover it, so the note now says so and points at [[Learning Curve]] as the empirical substitute the book does offer.

## Open questions

- `SGDRegressor` has no note of its own while [[Stochastic Gradient Descent Classifier]] does. Chapter 3 was about the classifier, and chapter 4 reaches for the regressor only twice, as a demonstration, never discussing it as a model, so the optimizer got the note ([[Stochastic Gradient Descent]]) and the regressor got aliases and code. Is that asymmetry right, or does `SGDRegressor` earn a note of its own once a later chapter leans on it?
- `Optimization/` went from one note to ten in a single chapter and still has no index note, on the precedent that `Evaluation/`, `Data/` and `Composition/` have none either and are described instead by a paragraph in [[Machine Learning]]. At what size does a folder earn an index rather than a paragraph?
- The five-route comparison table in [[Linear Regression]] was verified cell by cell against the scikit-learn 1.6 documentation and the chapter notebook's exercise answers, but HOML's own table 4-1 could not be reached. The one cell left to eyeball against the printed book is batch gradient descent's hyperparameter count of 2. It is 2 whether Géron counts the iteration budget as one parameter or as a `tol` and `max_iter` pair, so nothing turns on the answer, but it is a claim unverified against the source that states it.
- Probability calibration still has no note. Chapter 3 raised the gap between a score and a probability, and chapter 4 sharpens the need rather than closing it: [[Log Loss]] is a strictly proper scoring rule, [[Softmax Regression]] returns probabilities that regularization flattens without moving the argmax, and nothing in the vault says how to check whether a predicted probability means what it says.
- The no free lunch theorem is now the only chapter 1 topic with no note at all, regularization having been the other one until this chapter.
- Whether versicolor is linearly separable from virginica in all four [[Iris]] dimensions is asserted by both the scikit-learn and the UCI descriptions and could not be proved here. Only the two-dimensional petal-plane result is measured, where the best possible straight line still misclassifies 3 of those 100 rows. What settles the four-dimensional case is a linear program, which is exactly the margin machinery chapter 5 supplies.
- Second-order methods are named nowhere. Every optimizer in this chapter is first-order, using the [[Gradient]] and nothing beyond it, and the reason a Hessian is not used to pick the step is never given. Chapter 10 and 11's optimizer survey is the first place that could answer it.

Settled here, having been asked earlier:

- The learning rate against the gradient descent step size, asked by [[HOML Ch01 The Machine Learning Landscape]] and left standing again by [[HOML Ch03 Classification]]. They are one knob. In [[Stochastic Gradient Descent]] the single number $\eta$ multiplies the gradient of one instance's loss, and a squared-error update on one instance collapses to $\theta \leftarrow (1 - 2\eta)\theta + 2\eta\,y_i$, an exponential moving average whose effective memory is about $1/(2\eta)$ instances. So "how far the parameters move per step" and "how fast the system forgets what it saw before" are two readings of one coefficient. Two riders: in [[Batch Gradient Descent]] the adaptation reading has no content, because there is no stream, and a decaying [[Learning Schedule]] drives $\eta$ toward zero and so kills adaptivity over the life of the fit, which is why an online system tracking drift needs `learning_rate="constant"`. The derivation is in [[Learning Rate]].
- Chapter 1's regularization gap, which had been open since the first chapter note. [[Regularization]] now carries the category, with [[Ridge Regression]], [[Lasso Regression]], [[Elastic Net Regression]] and [[Early Stopping]] as its instances.
