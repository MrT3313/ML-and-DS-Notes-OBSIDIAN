---
note_kind: concept
aliases:
  - experiment tracking
  - experiment logging
  - run tracking
  - experiment artifact
  - experiment versioning
  - reproducibility
  - MLflow
  - Weights and Biases
  - W&B
up: "[[Machine Learning System]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Experiment tracking is the recording of what a training run does while it runs. A file the run produces on the way is called an **artifact** in that practice, meaning a loss curve, a validation loss plot, a log, an intermediate result of the model part way through fitting, and that is a narrower sense than the word carries everywhere else in this domain, where the artifact is the fitted model object itself. A reader who arrived for the fitted object wants [[Model]], which is the $A$ of [[Machine Learning System]]; nothing below is about that sense.

Versioning is the companion half, logging the details of a run so that the run can be recreated later or set beside another run and compared. The two began as separate practices and no longer ship separately, since the tools that started as experiment trackers have grown into version trackers as well. What happens to a finished model afterwards, once it is a candidate for deployment and needs a name, a stage and a rollback target, is a model registry and is separate machinery owed to a later source.

## Formal statement

[[Machine Learning System]] claims that deployed behaviour is a joint function of the code, the training data and the fitted artifact, so that pinning the code alone does not determine the behaviour and each of the three has to be pinned on its own. A training run is where that pinning is either done or not done. What follows is the condition made checkable.

That note writes the fit as $A = \text{fit}(C, D)$, with $C$ the code and $D$ the training data. Naming the arguments $C$ hides is what turns the claim into a procedure:

$$A = \text{fit}(C, D, \lambda, s, E)$$

- $C$, the **code version**: one commit identifier covering the whole fitting path, the preprocessing and the [[Pipeline]] included, not the model file alone.
- $D$, the **data version**: one identifier resolving to the exact rows fitted on. A content hash of the tracked file, or a numbered version of a table. A row count, a byte size or a date is not such an identifier.
- $\lambda$, the **configuration**: every [[Hyperparameter]] and setting the fit reads, including the ones that were defaulted rather than written down, since a default that changes between library versions is an unrecorded input wearing the costume of a constant.
- $s$, the **seed or seeds**: every generator the run draws from, which is more than one as soon as the framework, the array library and the data loader each hold their own. See [[Random Seed]].
- $E$, the **execution environment**: library versions, and hardware, device count and thread count wherever they change the result.

**The invariant.** Let $x_1, \dots, x_5$ be those five arguments. A run is reproducible only if every one of them is recorded:

$$\text{reproducible}(r) \implies \bigwedge_{i=1}^{5} \text{recorded}(x_i)$$

The direction is necessity and the conjunction is the whole content. Each argument moves $A$ on its own, so an unrecorded one is a free variable and the weakest unrecorded input decides what the record is worth. Four of five pinned and the fifth free is not eighty percent reproducible; it is not reproducible, and the record is a document about a run nobody can perform again.

Two things the invariant does not say, and stating both is what keeps it from being a slogan.

- **A complete record is not sufficient.** Some nondeterminism survives it. Threads reducing floating point sums in whatever order they finish, and workers in a distributed fit presenting batches in whatever order they arrive, both return a different $A$ from the same five arguments. Breck, Cai, Nielsen, Salib and Sculley put this first among their infrastructure tests, in a rubric drawn from interviews with 36 production systems at Google ("The ML Test Score", *IEEE Big Data* 2017): "Ideally, training twice on the same data should produce two identical models", and then, plainly, "Unfortunately, model training is often not reproducible in practice, especially when working with non-convex methods such as deep learning or even random forests." Determinism is therefore a property a run has to be built to have, rather than one a record confers on it.
- **An incomplete record is sometimes enough anyway.** A fit with no stochastic component repeats itself with $s$ unrecorded, because it never read $s$. That is not a counterexample; it narrows the claim correctly. The list is the set of inputs the fit actually reads, and an input the fit does not read costs nothing to omit and nothing to record. Deciding which is which by inspection is how runs become irreproducible, so the list is recorded whole.

Stated so it can be found false: take a run whose record omits exactly one of the five, re-execute it, and compare the returned $A$ against the stored one. The invariant predicts they can differ, and one pair that differs is enough to show the omitted argument was load-bearing. This is what makes reproducibility a testable property rather than an aspiration. That it is testable does not make it tested: of the twenty eight tests in the rubric above, drawn from thirty six real systems, none was implemented by more than eighty percent of the teams surveyed.

The word itself carries three distinct degrees and they are worth keeping apart, in Gundersen and Kjensmo's vocabulary (*AAAI* 2018, from a survey of 400 papers across IJCAI and AAAI in which between a fifth and a third of the variables they looked for were documented and no paper documented all of them). A result is **experiment reproducible** when the same implementation on the same data returns the same result, **data reproducible** when a different implementation on the same data returns the same result, and **method reproducible** when a different implementation on different data returns consistent results. A tracked run buys the first and only the first. The second and third are claims about the method rather than about the run, and no amount of logging produces them.

### What source control pins and what it leaves free

Version control is usually introduced as the ability to revert to an earlier state of the files and to let several people work on one codebase at once. Both are true, and both are true of every software project ever put in a repository, which is exactly why neither is the thing that matters here. The repository holds $C$. It does not hold $D$, which is too large for it and changes on a different schedule, and it does not hold $A$, which is a build output nobody wants to diff. So a project with immaculate source control and no other record has pinned one of five arguments, and the claim "this model came from commit `abc123`" is not a statement about any particular model.

That is the gap the practice exists to close, and it is the same gap the two neighbouring architecture requirements each point at without owning: [[Maintainability]] is about a later contributor being able to pick the system up, and what they have to be able to pick up here is not only the code, while [[Scalability]] reaches the same place from the other side, since artifact management stops being doable by hand at the first model in production rather than at the hundredth.

### What is tracked, and what each quantity diagnoses

Six families are worth recording, and the useful way to hold them is by the question each one answers rather than as a list of things to log.

| recorded quantity | what it diagnoses |
|---|---|
| loss on the training split and on each evaluation split, per step or per [[Epoch]] | whether the fit is progressing, plateauing or diverging. The gap opening between the two curves is [[Overfitting]] appearing, the step where the validation curve turns is what [[Early Stopping]] selects on, and the same pair plotted against training set size instead of against step is a [[Learning Curve]] answering a different question |
| the [[Performance Measure]] the project is actually judged on, computed on every split except the test set | whether a falling loss is moving the quantity anyone cares about. A surrogate loss and a reported metric can move in opposite directions, and only recording both catches it |
| one row per instance carrying the input, the prediction and the ground truth label | which instances are wrong rather than how many, which is the input [[Error Analysis]] reads, and the cheapest sanity check available that inputs and labels are still paired correctly |
| throughput: optimizer steps per second, or instances or tokens processed per second | whether a change made the fit slower. A throughput regression is invisible in every loss plot, because the loss plot has no time axis |
| system metrics: memory in use, CPU and GPU utilization | where the bottleneck is, and whether hardware that is being paid for is sitting idle |
| the trajectory of anything that moves during the fit: the [[Learning Rate]] under a schedule, gradient norms globally and per layer, weight norms | optimizer health. A gradient norm collapsing to zero or running away, or one layer's norm diverging from the rest, is a mechanism-level fault that the loss reports only as a flat line, and the per-layer split is what localizes it |

Recording more than seems necessary is the right default, and the argument is an asymmetry rather than a preference. Writing a scalar per step costs almost nothing and is paid once, while the alternative when something goes wrong is reconstructing the run and executing it again to obtain a number that could have been written down the first time. The fits worth investigating are the expensive ones, so the asymmetry is widest exactly where it is being relied on.

## Where it is used

[[Machine Learning System]] is the claim this carries out: its three parts are pinned separately or they are not pinned at all, and a run is repeatable only when all three are on the record. [[Random Seed]] is the argument $s$ under its own name, and it already states the thing a seed does not buy, that reproducible is not the same as correct. [[Model Debugging]] is the consumer: a bug that cannot be reproduced cannot be bisected, and validating a fix means running the fit again and comparing, which is a comparison only if the two runs differ in the one thing that was changed. [[Hyperparameter]] is most of $\lambda$, and [[Grid Search]] and [[Randomized Search]] are where a single tuning job generates tens or hundreds of runs at once, which is the point at which holding the comparison in a notebook stops working.

[[Maintainability]] and [[Scalability]] both ask for this and neither owns it, for the reason given above. [[MLOps]] is where the deployed half of the subject lives, monitoring, retraining cadence and rollback, and the record made here is what a rollback needs in order to have somewhere to roll back to. [[Machine Learning Project Lifecycle]] places the activity: it is phase 3, model development, and the reason the loop's back edges from phases 5 and 6 are affordable at all, since returning to a build phase means returning to a specific run rather than to a general intention.

There is no canonical paper here, and that absence is itself the finding. The practice was assembled from tooling rather than published as a result, so the primary sources are the tools' own current documentation plus three papers that each supply one piece: Sculley and colleagues on hidden technical debt in machine learning systems (*NeurIPS* 2015) for why the surround costs more than the model, Breck and colleagues' rubric above for reproducibility as something that gets tested and fails, and, for the vocabulary, Gundersen and Kjensmo above alongside Pineau and colleagues' report on the NeurIPS 2019 reproducibility programme (*JMLR* 22(164), 2021), whose machine learning reproducibility checklist is the closest thing the field has to an agreed statement of what a record should contain, and whose code submission rate rose from under half to close to three quarters of accepted papers in one year.

### In scikit-learn

scikit-learn 1.6, with NumPy 2.x and pandas 2.x, has **no experiment tracking and no versioning of any kind**. There is no run, no run identifier, no log, no history and no comparison view, and no plan for one; the library's scope ends at estimators and the tools for fitting and scoring them.

What the pinned stack has instead is `joblib` persistence, which writes a fitted estimator to a file, beside a `cv_results_` table that a search leaves on the search object in memory and that disappears with the process. Neither is a tracked experiment and presenting them as one is the mistake this paragraph exists to prevent: a pickle records $A$ and nothing about how $A$ came to be, and `cv_results_` records $\lambda$ and the scores for one search and nothing about $C$, $D$, $s$ or $E$.

The library says so itself, in the terms of the invariant above. Its model persistence page warns that loading a model fitted under a different version of scikit-learn is unsupported and inadvisable, raising `InconsistentVersionWarning` when it happens, and it lists what has to be stored alongside the pickle if the model is to be rebuilt later: the training data, as a reference to an immutable snapshot; the Python source that generated the model; the versions of scikit-learn and its dependencies; and the cross-validation score obtained on the training data. That list is $D$, $C$, $E$ and a recorded result, written by the library that declines to record any of them for you.

### Tools that implement it

Named as instances rather than described as products, and every line dated, because a roster of commercial tools goes stale faster than anything else in this vault. Tooling for machine learning operations in depth, this market included, belongs to [[DMLS]] chapter 10 and is not settled here.

Verified as of September 2026:

- **MLflow** is the open source tracking server and the default answer when the record has to be self-hosted. It keeps runs under an experiment identifier with parameters, metrics and artifacts attached, and it carries a model registry alongside the tracker, which is the other half of the subject in the same deployment. Version 3.15.0 was released in July 2026.
- **Weights and Biases** is the hosted form, a client library that streams metrics and artifacts to a service that holds the comparison interface over runs, and it is the one most often reached for on large training jobs. It was acquired by CoreWeave in May 2025 and is now sold as part of that company's cloud platform rather than as an independent service, which is a fact about who owns the roadmap rather than about the product today.
- **DVC** comes at it from the data side: it puts a content hash of the tracked file or directory into a small metafile that goes into git while the bytes go to a cache or a remote, which is $D$ pinned by the same mechanism that pins $C$. DVCLive is the logging companion that makes a run's metrics and plots part of the same record.
- **Comet** and **Aim** are the standing alternatives, the first hosted and paid, the second open source and self-hosted, and both are worth knowing about mainly because the first choice on this list is not always available.
- **Neptune** was on every version of this list until recently and is the sharpest available argument for dating every line above. Its acquisition by OpenAI was announced in December 2025 and its hosted service shut down on 4 March 2026, with the company's own notice stating that "Any remaining hosted data will be securely and irreversibly deleted as part of the shutdown."
