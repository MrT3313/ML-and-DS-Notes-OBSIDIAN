---
note_kind: method
aliases:
  - model debugging
  - debugging
  - debugging machine learning
  - overfit a single batch
  - overfit one batch
up: "[[Machine Learning System]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Model debugging is finding out why a fitted model is not doing what it should, and it is a different activity from debugging ordinary software for three reasons that compound.

- **It fails silently.** Nothing raises, nothing crashes, no service reports an error, and the model goes on returning predictions of the same shape and the same type as before. The only evidence is that the numbers are worse than they should be, and how much worse they should be is usually not known. Goodfellow, Bengio and Courville put the general form of this plainly: "In most cases, we do not know a priori what the intended behavior of the algorithm is" (*Deep Learning*, MIT Press 2016, section 11.5). A test on ordinary software asserts a stated output for a stated input; there is no such assertion available here, which is why the failure has to be hunted rather than caught.
- **Validating a fix is slow.** Confirming that a change repaired anything means refitting and re-evaluating, so the loop that would take seconds on a unit test takes as long as a training run. That cost sets the whole strategy: the point of every technique below is to get the number of full fits down, because each one is the price of a single hypothesis.
- **The failure surface spans everything at once.** Data, features, labels, algorithm, code and infrastructure interact, so a symptom in one of them is routinely caused by another, and the usual move of isolating a component fails because the components are not separable in the way a module boundary is separable. This is the same fact [[Machine Learning System]] states as its central claim, met from the maintenance side: behaviour is a joint function of code, data and the fitted artifact, so there is no single place to put a breakpoint.

Reach for this when a model trains without error and scores badly, when a score moves for no reason anybody can name, or before either happens, since the practice is preventive as well as curative. The curative half is the protocol below. The preventive half is habits that keep bugs from proliferating in the first place: every experiment recorded so a result can be repeated and compared ([[Experiment Tracking]]), every preprocessing step inside a [[Pipeline]] so a transform cannot be fitted across a split by accident, and the model specification under review and in a repository like any other code, which Breck, Cai, Nielsen, Salib and Sculley make the first of their model development tests because, in their words, "it's crucial to know the exact code that was run to produce a given learned model" ("The ML Test Score", *IEEE Big Data* 2017).

### Five things that make a model fail

Worth holding as a fixed list, because the first move in any debugging session is deciding which of them is in play, and the list is short enough to run through.

1. **Theoretical constraints.** Every model comes with assumptions about the data and the features it is given, and it fails when the data it learns from does not conform to them. The assumptions are the model's [[Inductive Bias]], and the failure is not a bug in anything: the code is correct, the fit converged, and the model was the wrong model.
2. **Poor implementation of the model.** The family is a good fit and the code that implements it has bugs. This is the only one of the five that is a defect in the ordinary sense, and it is the one the protocol below is sharpest at finding.
3. **Poor choice of hyperparameters.** With one model, one setting gives a state-of-the-art result and another causes the fit never to converge at all. Nothing about the code differs between the two. See [[Hyperparameter]].
4. **Data problems.** Instances and labels paired incorrectly, noisy labels, features normalized with statistics that are out of date, and a long tail of others. [[Poor-Quality Data]] is the standing account of the file-level half of this.
5. **Poor choice of features.** Too many and the model overfits, or a column carries the answer and the fit learns something that will not exist at inference time, which are [[Overfitting]] and [[Data Leakage]] respectively. Too few and the feature set lacks the predictive power the task needs.

Only the second is found by reading code. The other four are found by running something and reading what comes back, which is why the answer to this subject is a protocol rather than a debugger.

## Algorithm

Three techniques, in the order they are worth doing. The first two have notes of their own and are named here only for their place in the sequence; the third is the one developed below.

1. **Start simple and add components one at a time.** Begin with the simplest model that could work and grow it, so that every component arrives with a before and an after. [[Model Selection]] carries this as one of its judgements about how a candidate set gets built.
2. **Fix the seeds before comparing anything.** [[Random Seed]] carries what a seed is, what fixing it buys, and what it does not.
3. **Overfit a single batch.** The rest of this section.

### Overfitting a single batch

The premise is that fitting a handful of rows exactly is the easiest thing the machinery can be asked to do. A model with enough capacity to be worth running at all has enough capacity to memorize thirty rows, so if it cannot, something is broken rather than merely hard.

1. **Draw a deliberately tiny subset** of the training data. A few rows, on the order of two to thirty two, and for a classifier at least one row of every class. Draw them through the real loader and the real [[Pipeline]], not by hand, because the plumbing is part of what is under test.
2. **Turn off everything whose job is to prevent overfitting.** [[Regularization]] penalties, dropout, [[Data Augmentation]], [[Early Stopping]]. Each of them exists to stop exactly what is being attempted here, so leaving one on makes a healthy pipeline fail the test.
3. **Fit on those rows** for as many iterations as it takes, with no cap that fires before the loss has flattened.
4. **Evaluate on those same rows.** Not a held-out slice, not a fold. The rows that were fitted on.
5. **Read the pass condition: the loss reaches its floor.** For a classifier, training loss near zero and training accuracy at $1.0$ on those rows. For a regressor, training error near zero. Anything meaningfully short of the floor is a failure.
6. **On a failure, stop and find the bug before touching anything about generalization.** On a pass, go on to the real fit.

### What a pass proves and what it does not

A **failure localizes the fault to the machinery**, and to one of four places: the model, the loss, the optimizer, or the data plumbing that pairs $\mathbf{x}$ with $y$. Goodfellow, Bengio and Courville give the test as one of their debugging strategies and state the inference directly: "Usually if you cannot train a classifier to correctly label a single example, an autoencoder to successfully reproduce a single example with high fidelity, or a generative model to consistently emit samples resembling a single example, there is a software defect preventing successful optimization" (*Deep Learning*, 2016, section 11.5). Breck and colleagues arrive at the same test from the testing side, recommending among their unit tests for model specification code that one "purposefully train a model for overfitting: if one can get a model to effectively memorize its training data, then that provides some confidence that learning reliably happens" (*IEEE Big Data* 2017). Where the technique is citable from is worth being exact about. [[DMLS Ch06 Model Development and Offline Evaluation|DMLS chapter 6]]'s own resource list sends the reader to Karpathy's "A Recipe for Training Neural Networks" (2019) for the training and debugging recipe, and that is a blog post rather than a reviewed source. So the pointer and the citation are two different documents: the two above are where this note takes the technique from, and the pointer is recorded rather than repeated as though it were the source.

A **pass rules out a broken pipeline and nothing else**. It says the optimizer can drive the loss down, that the gradient reaches the parameters, that the labels line up with the inputs, and that the loss is the one intended. It says nothing whatever about [[Generalization]]. A model that overfits eight rows may still be the wrong model, may still be given the wrong features, and will very often be worse than a [[Baseline Model]] on real data. Those are the other four causes above and this test does not touch them.

**Evaluating on the training rows is correct here and nowhere else.** Every other note in the vault that touches this forbids it, and the prohibitions are right: [[Error Analysis]] requires out-of-fold predictions because predictions on fitted rows describe memorisation, [[Testing Set]] exists to keep an honest estimate away from the fit, [[Data Leakage]] and [[Data Snooping Bias]] name the two ways the boundary gets crossed, and [[Overfitting]] is defined by the gap between the two sides of it. None of that is being suspended. What changes is what the number is being read as. Everywhere else the training score is read as an estimate of generalization, and it is a bad one. Here it is read as a check that the optimizer can reach the floor at all, and for that reading the fitted rows are the only rows that answer the question, because a held-out score cannot distinguish a broken optimizer from a hard problem.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| subset size | $n$ | one batch, on the order of 2 to 32 rows | a pass becomes stronger evidence, since more distinctions had to be represented to reach the floor, and a failure becomes weaker evidence, since at large $n$ a model can genuinely lack the capacity to fit the sample or the sample can carry contradictory labels, and neither of those is a bug | start at the smallest $n$ holding at least one row of each class, so the loss has something to separate. Raise it only after a pass, to make the next pass mean more, and never far enough that a failure stops being diagnostic |
| iteration budget | $T$ | high enough that the loss visibly flattens | fewer false failures, since a working pipeline stopped early looks identical to a broken one, at the cost of time | raise it until the loss curve is flat rather than still falling, then read the pass condition. A budget that has to be raised repeatedly is itself a finding about the [[Learning Rate]] |

Two things sit outside the table on purpose. The learning rate and the regularization switches decide whether the batch is fitted at all, but they belong to the fit being debugged rather than to the protocol, and the protocol's instruction about the second group is fixed rather than tunable: turn them off. The [[Random Seed]] is pinned rather than tuned, for the reason that note gives.

## Failure modes

- **Reading a pass as evidence about generalization.** The test is passed by any pipeline that works, including one whose model is hopeless for the task. Treating a green result as permission to stop is how a correctly wired implementation of the wrong model gets trained for a week.
- **Leaving a regularizer on.** A weight penalty, dropout, augmentation or [[Early Stopping]] each stop the loss reaching its floor by design, so the run fails and a bug hunt begins for a bug that does not exist. scikit-learn's default is the standing trap: `LogisticRegression` applies an L2 penalty at `C=1.0` unless told otherwise.
- **A subset containing one class.** A classifier that always returns the same label reaches near-zero loss on a single-class subset without the machinery doing anything at all, so the test passes on precisely the kind of broken implementation it exists to catch. The remedy is in step 1, at least one row of every class.
- **Building the tiny batch by hand.** A handwritten array bypasses the loader and the transformers, and misaligned inputs and labels in the loader are the most common plumbing bug this test is supposed to find. Bypassing the plumbing under test makes the test a test of nothing.
- **Concluding from a failure without bisecting.** A failure says a fault exists somewhere among the model, the loss, the optimizer and the plumbing, and says nothing about which. The next move is technique 1, removing components until the test passes, and the component whose removal fixed it is the answer.
- **Declaring a fix without re-running.** Because validating a fix costs a full fit, the temptation is to accept that the code now looks correct. A fix that has not been re-run against the failing test is a hypothesis, and in this subject a hypothesis that looks correct and is wrong is the normal case rather than the exception.
- **Debugging a run that cannot be repeated.** Two runs that differ in an unrecorded input differ in ways no comparison can attribute, so the before and after that technique 1 depends on is not a comparison at all. [[Experiment Tracking]] is the precondition, not an optional tidiness.

## Implementation

scikit-learn 1.6 has no built-in check of this kind, which is consistent with it having no [[Experiment Tracking]] either: the library fits and scores estimators and leaves the surrounding discipline to the caller. What the test costs is about ten lines, and the value is in where they are pointed rather than in what they contain.

```python
import numpy as np
from sklearn.metrics import accuracy_score, log_loss

# 1. a tiny stratified batch, drawn from the real training arrays
rng = np.random.default_rng(42)
idx = np.concatenate([rng.choice(np.flatnonzero(y_train == c), size=4, replace=False)
                      for c in np.unique(y_train)])
X_tiny, y_tiny = X_train.iloc[idx], y_train.iloc[idx]

# 2 and 3. fit the WHOLE pipeline, preprocessing included, so the plumbing is
# exercised rather than bypassed. C=1e6 is how the L2 penalty is switched off.
pipe.set_params(clf__C=1e6, clf__max_iter=10_000)
pipe.fit(X_tiny, y_tiny)

# 4 and 5. scored on the rows it was fitted on, which is correct only here
print(accuracy_score(y_tiny, pipe.predict(X_tiny)))          # want 1.0
print(log_loss(y_tiny, pipe.predict_proba(X_tiny)))          # want near 0
```

Three details carry the weight. `X_train` rather than a handwritten array, so the transformers run; the `.iloc` assumes a DataFrame and becomes plain `X_train[idx]` on a NumPy array. `C=1e6` rather than the default `C=1.0`, because the L2 penalty is on by default and would hold the loss off its floor, and the double underscore addresses the pipeline step named `clf`; the equivalent switch is `penalty=None` on `SGDClassifier` and `SGDRegressor`, where `early_stopping=False` is also required and is already the default. And `max_iter` raised far above the default `100`, because a `ConvergenceWarning` and a genuine failure look the same in the printed numbers.

The test becomes a standing guard rather than a one-off by wrapping the two prints in assertions and running them on a fixed tiny sample in continuous integration, which is the shape the same rubric's infrastructure test for unit testing model specification code asks for. That turns an expensive question, whether the training code still works, into a cheap one that runs on every commit.

One honest limit on the pinned stack. The premise is that the model has the capacity to memorize $n$ rows, and a low-capacity model on a small feature set may not: a plain [[Logistic Regression]] on three features cannot separate an arbitrary sixteen rows, and a failure there is a true fact about the model rather than a bug. That is the reason $n$ is kept small, and the reason the technique is sharpest on models with capacity to spare.
