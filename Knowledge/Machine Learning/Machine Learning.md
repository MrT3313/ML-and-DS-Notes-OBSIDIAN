---
note_kind: index
aliases:
  - ML
up: "[[Home]]"
---

The root of the machine learning domain. Everything under `Knowledge/Machine Learning/` hangs off this note. Its job is to hold the definitions the rest of the vault leans on, and to name the areas so nothing gets filed by accident.

> [!quote] Definition(s)
>Machine learning is the science (and art) of programming computers so they can learn from data. ~[[HOML Ch01 The Machine Learning Landscape|HOML Chapter 1]]
>
>Machine Learning is the field of study that gives computers the ability to learn without being explicitly programmed. ~Arthur Samuel, 1959
> 
>A computer program is said to learn from experience $E$ with respect to some task $T$ and some performance measure $P$, if its performance on $T$, as measured by $P$, improves with experience $E$. ~Tom Mitchell, 1977

Mitchell's $E$, $T$ and $P$ are the three hooks the rest of the domain hangs on: $E$ is the [[Training Set]], $T$ is one of the tasks listed under Areas below, and $P$ is the [[Performance Measure]].

## Areas

- **`_Foundations/`** the framework-independent vocabulary every other note assumes: [[Model]], [[Feature]], [[Hyperparameter]], [[Training Set]], [[Training Instance]].
- **`Paradigms/`** the three independent axes a system can be placed on. Supervision ([[Supervised Learning]] through [[Unsupervised Learning]]), training regime ([[Batch Learning]] versus [[Online Learning]]), and representation ([[Instance-Based Learning]] versus [[Model-Based Learning]]).
- **`Tasks/`** what the system is asked to do: [[Classification]], [[Regression]], [[Clustering]], [[Dimensionality Reduction]], [[Anomaly Detection]], [[Novelty Detection]], [[Association Rule Learning]]. `Classification/` splits the first of those by the shape of its target: [[Binary Classification]] for two exclusive classes, [[Multiclass Classification]] for one label out of three or more, [[Multilabel Classification]] for several yes-or-no labels carried at once, and [[Multioutput Classification]] for several targets each with classes of its own, which is the general case the other three are constrained versions of.
- **`Models/`** the algorithm families, one folder per family. `Linear/` holds [[Linear Regression]], [[Logistic Regression]] and the [[Stochastic Gradient Descent Classifier]], a linear classifier trained one instance at a time whose `loss` argument decides which linear model is actually being fitted; `Ensembles/` holds [[Ensemble Learning]], the idea rather than any method built on it.
- **`Objectives/`** what training optimizes: [[Performance Measure]] and its two sign conventions, [[Cost Function]] and [[Utility Function]]. `Metrics/` holds the measures themselves, [[Root Mean Squared Error]] and [[Mean Absolute Error]] for regression, and for classification the [[Confusion Matrix]], which is the table the rest are read off. [[Accuracy]], [[Precision]], [[Recall]] and [[F1 Score]] are each a ratio of its cells, and [[Precision-Recall Tradeoff]] and [[ROC Curve]] are what you get by sweeping the decision threshold and taking a fresh pair of those ratios at every setting.
- **`Optimization/`** the mechanics of fitting. [[Learning Rate]] today, gradient descent and regularization from chapter 4.
- **`Evaluation/`** estimating [[Generalization]] honestly: [[Testing Set]], [[Holdout Validation]], [[Cross-Validation]], [[Model Selection]], the two failures, [[Overfitting]] and [[Underfitting]], and [[Data Snooping Bias]], the way a test set quietly stops being one. [[Baseline Model]] sits at the front of all of it, the floor a candidate has to clear before it is a candidate at all, and [[Error Analysis]] at the back, reading which mistakes a model makes rather than how many, so the next thing to fix is chosen rather than guessed. `Hyperparameter Search/` automates the choosing, through [[Grid Search]] and [[Randomized Search]].
- **`Data/`** everything about the input. At the top sit [[Feature Engineering]], deciding what the model actually sees, [[Data Augmentation]], growing the training set with label-preserving transformations of what is already in it, and [[Open Data Repositories]], where a dataset comes from in the first place. `Challenges/` holds the seven ways data defeats a model, [[Class Imbalance]] among them, a label whose classes are not present in comparable numbers, so predicting the majority beats learning anything. `Sampling/` carves a dataset up: [[Random Sampling]], [[Stratified Sampling]], and the [[Random Seed]] that makes either one reproducible. `Exploration/` is the looking that comes before any fitting: [[Exploratory Data Analysis]], [[Correlation]], [[Skewed Data]]. `Preprocessing/` falls into three jobs. Fill the gaps, [[Missing Value Imputation]]. Turn categories into numbers, [[Ordinal Encoding]] and [[One-Hot Encoding]]. Put columns on comparable ranges, [[Feature Scaling]] and its two forms [[Min-Max Scaling]] and [[Standardization]], with [[Feature Distribution Transformation]] for columns that need reshaping first and [[Target Scaling]] for the label.
- **`Composition/`** the plumbing that combines estimators into one estimator, acting on estimators rather than on data. [[Scikit-Learn Estimator API]] is the contract the other six rest on, and its estimator, transformer and predictor taxonomy is what the `## Implementation` sections across the vault assume, not these six alone. [[Pipeline]] chains steps into a single fitted object and is what guarantees each step sees training folds only, [[Column Transformer]] routes named column subsets to their own branch and concatenates the results, and [[Custom Transformer]] is how an idea of your own earns the same interface, and so the same safety, as a built-in. The same job on a different axis is done by the meta-estimators, which wrap one estimator to give it a capability it does not natively have: [[One-versus-Rest]] and [[One-versus-One]] hand a two-class algorithm a multiclass target, the first by fitting one classifier per class and the second one per pair of classes, and [[Classifier Chain]] hands a single-label algorithm a multilabel one, by fitting a classifier per label in a fixed order and feeding each of them the labels already decided.

Operational concerns sit outside this domain, in `Knowledge/MLOps/`, indexed at [[MLOps]]. [[Model Rot]] is the first of them. The mathematics these notes lean on sits outside it too, in `Knowledge/Mathematics/`, indexed at [[Mathematics]] and holding [[Lp Norm]], [[Triangle Inequality]] and [[Moment]] so far.

## What is missing

- Reinforcement learning has no note and no folder; HOML chapter 1 covers it.
- Regularization and the no free lunch theorem are named in chapter 1 and written nowhere.
- `Models/` still holds one real family and a concept note. `Linear/` gained a classifier with chapter 3 but nothing else, and `Ensembles/` carries [[Ensemble Learning]] alone, with no ensemble method under it yet; trees, kernel methods and neural networks all arrive in later chapters.
- The models chapter 2 actually fits went unwritten. DecisionTreeRegressor, RandomForestRegressor and SVR all appear in its worked example, and chapters 6, 7 and 5 own them.
- Chapter 3 fits three models it produced no note for, and only two of them are owed one later. `SVC` carries its multiclass and chained demonstrations and a random forest supplies the comparison case on every curve in [[ROC Curve]], which chapters 5 and 7 own. The k-nearest-neighbours classifier behind its [[Multilabel Classification]] and [[Multioutput Classification]] examples is owned by no HOML chapter at all, so nothing is scheduled to bring it and there is no neighbours folder waiting for it.
- Probability calibration has no note, although chapter 3 leans on the gap between a score and a probability repeatedly: the [[Stochastic Gradient Descent Classifier]] offers no probability under its default loss, [[One-versus-Rest]] compares scores from classifiers that were never made comparable, and [[ROC Curve]] has to read one model through `decision_function` and the other through `predict_proba` to put them on the same axes.
- Feature extraction is the one branch of [[Feature Engineering]] with no note of its own.
- Launch, monitor and maintain, the last section of chapter 2, produced nothing. It belongs in `Knowledge/MLOps/` beside [[Model Rot]], and it is the largest single gap chapter 2 left.
- Framing the business objective and the machine learning project checklist, which open chapter 2, likewise produced no note.
