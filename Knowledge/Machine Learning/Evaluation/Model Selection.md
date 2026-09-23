---
note_kind: concept
aliases:
  - hyperparameter tuning
  - hyperparameter search
  - model comparison
up: "[[Generalization]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Model selection is choosing between candidate models, including the same model family under different [[Hyperparameter]] settings, by comparing their estimated [[Generalization]] error. It has to be done on data the candidates did not train on and that is not the [[Testing Set]].

## Formal statement

Given candidates $c \in \mathcal{C}$ with validation scores $\hat{\mathcal{L}}_{\text{val}}(c)$,

$$c^{*} = \arg\min_{c \in \mathcal{C}} \hat{\mathcal{L}}_{\text{val}}(c)$$

The estimate comes from [[Holdout Validation]] or [[Cross-Validation]]. Because the test set was not touched, its score for $c^{*}$ remains an unbiased generalization estimate.

$\hat{\mathcal{L}}_{\text{val}}$ is one number and a selection reads more than one. What a candidate costs to obtain and to run enters alongside how well it scores: how many labels it needs before it works at all, how much compute and wall clock one fit takes, what a single prediction costs in inference latency, and whether a prediction can be explained. These are not tie-breakers applied after the $\arg\min$, they are components of the object being compared, which is what makes selection a decision rather than the evaluation of a formula. As soon as two of them move in opposite directions there is no scalar left to minimize and no complete ordering to minimize it over, which is the structure [[Production Machine Learning]] states exactly: Pareto dominance, a set of non-dominated candidates rather than an optimum, and a preference supplied from outside the comparison to pick one of them.

One of those criteria has to be stated carefully rather than asserted, because it is a claim about a whole field. Non-neural models are commonly held to be more explainable than neural networks, the reason given being that you can read off which features contributed most to a prediction. Lipton's "The Mythos of Model Interpretability" (2016) questions that assertion directly, and following it keeps this note consistent with the treatment of the term in [[Production Machine Learning]]: the claim holds for *algorithmic transparency*, where following the fitted mechanism is uncontroversially easier for a linear model or a shallow tree, and it does not survive the other notions. High-dimensional features cost a linear model its simulatability and heavily engineered features cost it its decomposability, while a network operating on raw or lightly processed inputs has features that are individually meaningful and a learned representation that can be visualized or clustered, which is an advantage for post-hoc explanation rather than a deficit. So the criterion is usable once it names which notion of interpretability the project needs, and the bare comparison between families is not, which is the qualification [[Production Machine Learning]] already attaches to the word.

### Conditions on a valid comparison

An $\arg\min$ over candidate scores means something only under conditions, and the conditions are worth stating as conditions because each one can be found false about a comparison somebody has already run and reported.

- **The same measure on the same splits.** Every candidate is scored by the same [[Performance Measure]] on the same folds, so that the numbers are differences between candidates rather than differences between measurements. A candidate scored on its own splits is not in the comparison, it is beside it, and [[Baseline Model]] is held to this requirement for the same reason the real candidates are: a floor measured differently from what stands on it proves nothing.
- **Comparable tuning effort.** Each candidate gets a search of comparable size before its score counts. A model family given fifty [[Hyperparameter]] configurations against one given three has not been shown to be better, it has been shown to have been worked on harder, and the difference between those two statements is the whole content of the comparison.
- **The conclusion is indexed to what it was measured on.** "$A$ beats $B$" is a claim about $A$ and $B$ on this dataset, at this sample size, under this preprocessing, at this budget. It is not a claim about $A$ and $B$. A better-than result can be true in one context and false in another without either measurement being wrong, so a comparison reported without its context has had its content removed rather than generalized.

The comparable-effort condition fails through a mechanism worth naming, because the mechanism is not carelessness. An engineer who finds one architecture more interesting spends longer on it, tries more features against it and more configurations of it, and therefore gets more out of it, so the measured winner is partly a function of who ran the experiment and what they were hoping for. Nothing about the resulting number looks wrong. [[Experiment Tracking]] is what turns the condition from an intention into something auditable, since the only way to check that two candidates got comparable searches is a record of how many configurations each one actually received.

These failures have been measured rather than merely argued. Musgrave, Belongie and Lim, "A Metric Learning Reality Check" (ECCV 2020), audited four years of deep metric learning and found three of them at once: the network architecture was not held constant across papers, so part of the reported gain was the backbone (their own table puts ImageNet-pretrained Recall@1 on CUB200 at $41.1$ for GoogleNet against $48.7$ for ResNet50 before any metric learning happens at all); baseline losses were not re-tuned but carried forward from a 2016 implementation with a badly chosen margin, which is how relative improvements over the contrastive loss came to exceed $100\%$; and most papers used no validation set, checking test accuracy at intervals during training and reporting the best, so model selection was run with direct test-set feedback. Under a protocol holding architecture, embedding dimension and augmentation fixed and tuning every method by [[Cross-Validation]] with Bayesian optimization, accuracy against publication year flattens to roughly a level line from 2006 to 2019. The reported progress was in the violated conditions.

## Where it is used

It is the loop that balances [[Overfitting]] against [[Underfitting]]. [[Grid Search]] and [[Randomized Search]] are the standard ways of enumerating $\mathcal{C}$, and [[AutoML]] is what the loop becomes when the enumeration itself is handed to a search algorithm, which changes the size and the cost of $\mathcal{C}$ and relaxes none of the conditions above. The failure Géron warns about: selecting on the test set, which makes the reported score a selection artifact.

$\arg\min$ over $\mathcal{C}$ answers only which candidate in the set is ahead, and it is silent about whether the set is the right set. The complementary diagnostic is a [[Learning Curve]], run on a single candidate rather than across them: it says whether the next move is more data or a different model family, which is the question you have to settle before enlarging $\mathcal{C}$ is worth the fits.

Two habits govern how $\mathcal{C}$ gets built in the first place, and both are about what a candidate costs before anybody scores it.

**"State of the art" is a fact about a benchmark.** A model called state of the art has beaten other models on a fixed dataset under one setup. That is evidence about the benchmark: it does not say the model is fast or cheap enough to run where you would run it, and it does not say it wins on your data, which the benchmark was not drawn from. [[Production Machine Learning]] carries the general form of the limit, that a leaderboard position is evidence about the leaderboard. The measured instance is Dacrema, Cremonesi and Jannach, "Are We Really Making Much Progress? A Worrying Analysis of Recent Neural Recommendation Approaches" (RecSys 2019): of $18$ neural recommendation methods from top-level conferences, $7$ could be reproduced with reasonable effort, $6$ of those $7$ were beaten by nearest-neighbour or graph-based heuristics, and the seventh cleared the simple baselines but not a well-tuned non-neural linear ranking method. The arguments this habit rests on are Sculley, Snoek, Wiltschko and Rahimi, "Winner's Curse? On Pace, Progress, and Empirical Rigor" (ICLR 2018 workshop), and Lipton and Steinhardt, "Troubling Trends in Machine Learning Scholarship" (2018).

**Start with the simplest candidate**, for three reasons that are worth keeping apart because each one holds without the others. Deploying something early validates that the prediction pipeline agrees with the training pipeline, which is a claim about plumbing that no amount of offline scoring can test. Adding complexity a step at a time keeps the model debuggable, because each change has one cause, which is the first move in [[Model Debugging]]. And the simplest candidate is the floor everything else is measured against, which is [[Baseline Model]].

Simplest is not the same as least effort, and the two come apart whenever an ecosystem does the work. A pretrained model can be the cheapest thing to get a score out of, the expensive part having been paid by somebody else and the weights shipped ready to load, while being the hardest thing to move away from later, since anything touching the architecture or the pretraining objective means paying for pretraining yourself. [[DMLS Ch06 Model Development and Offline Evaluation|DMLS chapter 6]] makes the point in 2022 with BERT as its example (Devlin, Chang, Lee and Toutanova, NAACL 2019). The example has dated and the shape of the claim has not: the cheap route to a first score is now an API call or a parameter-efficient fine-tune of a pretrained transformer, and the expensive route is still everything that reaches back into pretraining, so "simple to start with, hard to build on" describes a larger class of candidates now than it did when it was written about one model.

Trade-offs are what the selection criteria become once two of them move together. Inside a single classifier the standard pair is false positives against false negatives, which is not a choice between models at all but a choice of threshold, derived in [[Precision-Recall Tradeoff]]. Across candidates the standard pair is compute against accuracy: a more accurate model can demand a GPU rather than a CPU to return its prediction within the same latency, so the accuracy was bought with serving cost, and [[Production Machine Learning]] is where that cost is accounted for.
