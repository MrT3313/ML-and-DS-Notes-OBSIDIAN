---
note_kind: source
medium: book
chapter: 2
up: "[[HOML]]"
url: https://github.com/ageron/handson-ml3/blob/main/02_end_to_end_machine_learning_project.ipynb
aliases:
  - HOML Chapter 2
  - End-to-End Machine Learning Project
---

## Thesis

Chapter 2 is the whole workflow run once on California housing data, in the order you would actually run it: frame the business objective, get the data, put a test set out of reach before looking at anything, explore what is left, prepare it through a transformation that is fitted rather than hand-applied, shortlist a few models, tune the promising one, then evaluate once and ship. The argument the sequence makes is that the modelling is the small part. Almost every step is a place where held-out information can leak back into the model, and what keeps the final number honest is the discipline of estimating every transformation parameter on the training set alone and carrying the whole preparation inside one composable object that is refitted, searched over, and deployed as a unit. The chapter's second demonstration is that the raw columns understate what the data holds: combining attributes into ratios and reshaping skewed columns produces features that track the target far better than the columns they are built from, the ratio of bedrooms to rooms against the two near-useless counts behind it.

## Extracted

Framing and tooling:
- [[Scikit-Learn Estimator API]]

Getting the data:
- [[Open Data Repositories]]
- [[California Housing]]

Holding out the test set:
- [[Random Sampling]]
- [[Stratified Sampling]]
- [[Random Seed]]
- [[Data Snooping Bias]]

Exploring:
- [[Exploratory Data Analysis]]
- [[Correlation]]
- [[Skewed Data]]
- [[Moment]]

Preparing the data:
- [[Feature Engineering]]
- [[Missing Value Imputation]]
- [[Ordinal Encoding]]
- [[One-Hot Encoding]]

Scaling and distribution:
- [[Feature Scaling]]
- [[Min-Max Scaling]]
- [[Standardization]]
- [[Target Scaling]]
- [[Feature Distribution Transformation]]

Composing the preparation:
- [[Pipeline]]
- [[Column Transformer]]
- [[Custom Transformer]]

Error measures:
- [[Root Mean Squared Error]]
- [[Mean Absolute Error]]
- [[Lp Norm]]
- [[Triangle Inequality]]
- [[Mathematics]]

Tuning:
- [[Grid Search]]
- [[Randomized Search]]

Model families:
- [[Ensemble Learning]]

Amended, not produced, by this chapter:
- [[Testing Set]]
- [[Cross-Validation]]

Corrected by this chapter:
- [[Regression]]

## Open questions

- The chapter shortlists three models and none of them has a note. DecisionTreeRegressor is chapter 6's, RandomForestRegressor and the rest of the ensembles are chapter 7's, SVR is chapter 5's. [[Ensemble Learning]] is written as the argument for why averaging independent learners helps, with no mechanism in it on purpose, so bagging and boosting have somewhere to land when chapter 7 arrives.
- Launch, monitor and maintain is the last section of the chapter and it produced nothing at all, which made it the largest gap here. What else does `Knowledge/Machine Learning/Operations/` need before it is a real area: deployment targets, monitoring and alerting on live inputs, a retraining cadence, rollback, and where does the confidence interval on a shipped model's error belong? [[DMLS Ch01 Overview of Machine Learning Systems]] settles it partly. That folder no longer holds only its index and [[Model Rot]]: the setting itself is now stated, several parties wanting different things from one model whose data will not hold still, and the retraining cadence has a name and a research problem attached to it in [[Continual Learning]], keeping a deployed model fitted on data that arrives after it shipped without destroying what it already knew. Deployment targets have since arrived with [[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]]: [[Batch and Online Prediction]] for when a prediction is computed, [[Model Inference]] and [[Edge Computing]] for where, and [[Model Compression]] for what fits. Monitoring and alerting on live inputs are still absent, and DMLS chapter 8 is what owes them; rollback is named by no section heading in that book's table of contents, so it is unowned there rather than owed; and the confidence interval question is untouched.
- Framing the business objective and working the ML project checklist open the chapter and also produced no note. Is the checklist a note of its own under `__Meta/`, or is the checklist exactly what a chapter note like this one is already doing? That is an open question about this vault's structure rather than about the chapter. [[DMLS Ch02 Introduction to Machine Learning Systems Design]] settles it, as a split rather than as either option. The business objective earned [[Business Objective]], which does more than restate the step: it separates three quantities this chapter flattened into one, the differentiable thing training minimizes, the held-out number the trained model is scored by, and the business metric the decision is actually made on, and it states the order-preservation assumption joining the second to the third along with what fails when that assumption does not hold. The checklist resolves as neither option. The cycle the checklist walks earns a note, [[Machine Learning Project Lifecycle]], because it makes a structural claim that can be checked, that the feedback edges are exactly the complete bipartite connection from the two phases that learn something out of a running system to the two that build one, and a checklist makes no claim of that kind. The per-project checklist does not, because a chapter note like this one is already the record of what one project actually did, in order, and a `__Meta/` file listing the same steps in the abstract would be that record with the evidence taken out.
- Extraction is the one branch of [[Feature Engineering]]'s taxonomy with no note behind it. Which chapter actually brings it? Embeddings and feature extraction from text and images are Part II material, chapters 13 and 16, so the gap may sit open for a long while. [[DMLS Ch05 Feature Engineering]] **partly settles** it, and from the other book. [[Embedding]] now names the object: the vector a piece of data is represented by, the space shared by every vector the same algorithm produces for that type of data, and the discrete and continuous positional cases. The branch is no longer one with nothing behind it. What is still outstanding is the architectures that learn the mapping, and those remain [[HOML]] chapters 13 and 16.
- Grid search on the housing pipeline chose `n_clusters=15` and `max_features=6`, and 15 was the largest value the grid offered. An optimum sitting on the boundary of the search means the search was too narrow, and [[Randomized Search]] confirmed it by sampling `randint(3, 50)` and finding 45 clusters at a validation RMSE of about 42,000 against the grid's best of about 44,000. So how do you know in advance that a range is wide enough, and is there anything to do about a boundary optimum besides noticing it and re-running with a wider grid?
- Whether to split `Knowledge/Machine Learning/Data/Preprocessing/` was investigated here and deliberately deferred to chapter 13, so chapter 13 does not redo the work. The buckets stay thin until then: `Encoding` would sit at 2 notes until chapter 12, then gain 5 in chapter 13 alone. The question is not whether to split but on which axis: by what the step does (scaling, encoding, transformation), by stage (loading versus transforming), or by library (scikit-learn versus Keras). Chapter 13 introduces the last two axes.
