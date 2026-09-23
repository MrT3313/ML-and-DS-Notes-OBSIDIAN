---
note_kind: concept
aliases:
  - AutoML
  - automated machine learning
  - auto ML
  - soft AutoML
  - hard AutoML
  - neural architecture search
  - NAS
  - architecture search
  - learned optimizer
  - learned optimizers
  - auto-sklearn
  - Keras Tuner
  - Ray Tune
up: "[[Model Selection]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

AutoML is handing the search for a machine learning system over to a search algorithm, so that what a person supplies is the space of allowed systems, the score to optimize and the compute budget, rather than the system itself. It divides by how much is handed over. **Soft AutoML** fixes the shape of the model and searches only its [[Hyperparameter]] settings. **Hard AutoML** searches the shape: the building blocks are given and the algorithm decides how to combine them, which is architecture search, and at its furthest it searches the learning procedure too.

The soft half needs no new machinery, because it is the [[Hyperparameter]] search already inside [[Model Selection]], enumerated by [[Grid Search]] or [[Randomized Search]], and when blind sampling is too slow by the Bayesian optimization glossed under [[Randomized Search]], whose reference is Snoek, Larochelle and Adams, "Practical Bayesian Optimization of Machine Learning Algorithms" (NeurIPS 2012). What earns the separate name is the hard half.

## Formal statement

An AutoML problem is well posed when four things are fixed, and it is underspecified without any one of them.

1. A **search space** $\Lambda$, the set of candidate systems expressible at all. For soft AutoML this is a product of hyperparameter ranges; for hard AutoML it is a grammar of allowed architectures.
2. An **objective**, a held-out score $\hat{\mathcal{L}}_{\text{val}}$ estimated by [[Holdout Validation]] or [[Cross-Validation]], since the candidates are being compared on data none of them trained on.
3. A **budget** $B$, in fits, wall clock or accelerator hours. Without it the problem is a statement about an optimum rather than a procedure, because $\Lambda$ is normally too large to enumerate.
4. A **performance estimation strategy** $\widetilde{\mathcal{L}}$, a cheaper stand-in for $\hat{\mathcal{L}}_{\text{val}}$ that ranks a candidate without training it to convergence.

What the search actually solves is therefore

$$\lambda^{*} = \arg\min_{\lambda \in \Lambda} \widetilde{\mathcal{L}}_{\text{val}}(\lambda) \qquad \text{subject to} \qquad \sum_{\lambda \in S} c(\lambda) \le B$$

with $S \subseteq \Lambda$ the subset actually evaluated and $c(\lambda)$ the cost of evaluating one candidate. Two consequences follow, and both can be found false by measurement.

**The argmin is over the surrogate, not the score.** The returned candidate is optimal for $\widetilde{\mathcal{L}}$, and it is optimal for $\hat{\mathcal{L}}_{\text{val}}$ only to the extent that the two agree on the *ranking* of candidates. The absolute values need not agree at all, and a cheap estimator is allowed to be badly calibrated so long as it orders correctly. The check is direct: take the top $k$ candidates under $\widetilde{\mathcal{L}}$, train each to convergence, and measure the rank correlation between the two orderings. A strategy that fails it is spending the budget on a leaderboard of the wrong quantity.

**Automating the search makes an untouched [[Testing Set]] more necessary, not less.** The score reported for $\lambda^{*}$ is a minimum over $|S|$ noisy estimates, and a minimum over noise is optimistically biased. This is the same fact [[Grid Search]] attaches to `best_score_`, and the bias grows with $|S|$, which is precisely the quantity automation increases: a hand-written grid evaluates tens of candidates and an automated search evaluates thousands, so the gap between the winner's validation score and its true generalization error widens with the thing being sold as the improvement. Cawley and Talbot, "On Over-fitting in Model Selection and Subsequent Selection Bias in Performance Evaluation" (*JMLR* 11, 2010), is the treatment of the two-level version of this, selection bias on the outer estimate as well as overfitting on the inner one. The invariant that follows is checkable and easy to violate: nothing in the automated loop, including the performance estimation strategy and any early-stopping rule inside it, may read the test set, and if the loop is enlarged the test set has to be re-consulted to know what the enlargement bought.

The bill separates the two halves. Soft AutoML costs $|S| \cdot K$ fits of an ordinary estimator plus one refit, which is the formula [[Grid Search]] and [[Randomized Search]] derive. In hard AutoML a single evaluation of $c(\lambda)$ is a full training run of a neural network, so the budget is quoted in accelerator time rather than in fits: the reinforcement-learning search of Zoph and Le ran on 800 GPUs for three to four weeks.

### Neural architecture search

Architecture search decomposes into three independent choices, the decomposition used by Elsken, Metzen and Hutter in "Neural Architecture Search: A Survey" (*JMLR* 20, 2019), whose abstract categorises the existing work along exactly these three dimensions. Naming them separately is the useful part, because the three fail in different ways and are usually reported as though they were one method.

**The search space** says which architectures are expressible. It is where the human prior re-enters after apparently being removed: a space narrow enough that most of its members are good has had part of the answer written into it, and a result from such a space is partly a result about the space. This is [[Inductive Bias]] relocated from the model to the set the model is drawn from, and it is the reason two architecture searches are not comparable unless their spaces are.

**The search strategy** says how $\Lambda$ is traversed. Random sampling is the baseline, and the honest reading of it is not that it loses badly but that it costs too much for what it gives: in the case study reported by Real, Aggarwal, Huang and Le, random search reached roughly $4\%$ test error on CIFAR-10 against roughly $3.5\%$ for reinforcement learning and for evolution, a margin small enough that the argument against sampling blindly is the bill rather than the accuracy. Reinforcement learning is Zoph and Le, "Neural Architecture Search with Reinforcement Learning" (ICLR 2017), where a controller network emits a description of an architecture and is rewarded by the validation accuracy of the network that description builds. The evolutionary route is Real, Aggarwal, Huang and Le, "Regularized Evolution for Image Classifier Architecture Search" (AAAI 2019), which modifies tournament selection with a preference for younger genotypes and produced AmoebaNet-A; their evidence is that evolution reaches the same final accuracy as reinforcement learning with better anytime behaviour and smaller models, which matters exactly when the budget is short.

**The performance estimation strategy** decides whether either of the other two is affordable, and it is the component most often left implicit. It is anything cheaper than "train to convergence and score on held-out data": fewer epochs, a subset of the training data, a scaled-down copy of the candidate, weights inherited from a parent instead of a fresh initialization, or a learning curve extrapolated from its first few epochs. Every one of them buys budget by accepting a ranking that may differ from the true one, which is the first consequence above, stated as a design choice rather than as a risk. Successive halving, carried by [[Grid Search]] and [[Randomized Search]], is a member of this family applied to hyperparameters: the low-resource rounds are a cheap estimate used only to eliminate and never to report. Its references are Jamieson and Talwalkar, "Non-stochastic Best Arm Identification and Hyperparameter Optimization" (AISTATS 2016), and Li, Jamieson, DeSalvo, Rostamizadeh and Talwalkar, "Hyperband: A Novel Bandit-Based Approach to Hyperparameter Optimization" (*JMLR* 18, 2018).

### Learned optimizers

A training run has three parts, each of which can in principle be searched: the model, the learning procedure, and the algorithm that finds the parameter values minimizing the objective, which for a neural network is [[Gradient Descent]]. Hard AutoML taken to its limit replaces that third part with one that is itself fitted. Andrychowicz, Denil, Gómez Colmenarejo, Hoffman, Pfau, Schaul, Shillingford and de Freitas, "Learning to learn by gradient descent by gradient descent" (NeurIPS 2016), cast the design of an update rule as a learning problem: the rule is a recurrent network that reads gradients and emits parameter updates, and it is trained to minimize the loss of the network it is optimizing, which is what makes the title literal rather than a joke. [[DMLS Ch06 Model Development and Offline Evaluation|DMLS chapter 6]] names this direction rather than developing it, so what stands here is the shape of the idea and its primary reference; the hand-written update rules a learned optimizer would replace belong to [[Gradient Descent]].

### In scikit-learn

scikit-learn 1.6 with NumPy 2.x and pandas 2.x. **scikit-learn ships no AutoML.** It ships the search loops that soft AutoML automates, `GridSearchCV`, `RandomizedSearchCV`, `HalvingGridSearchCV` and `HalvingRandomSearchCV`, and in all four the candidates come from a space written out by hand as a dictionary of lists or of distributions. Nothing in the library proposes a candidate, a preprocessing step or an estimator that was not named in the call.

The distinction that is easiest to blur is successive halving. `HalvingGridSearchCV` and `HalvingRandomSearchCV` are a **search budget policy**: they change how a fixed budget is distributed across a candidate set, eliminating weak candidates while they are still cheap, and the candidate set is exactly the one that was handed in. That is the performance estimation component of the decomposition above and nothing else, so calling it AutoML claims the automation of model design on the strength of a resource schedule. The library's own related-projects page is consistent with this, filing automated machine learning outside itself under an "Auto-ML" heading, where auto-sklearn and TPOT both appear as third-party drop-in estimators.

## Where it is used

[[Model Selection]] is the decision this automates, and automating it changes none of its structure: the result is still an $\arg\min$ over one held-out score, and the conditions a valid comparison has to meet apply to every one of the thousands of comparisons an automated search makes rather than being relaxed by volume. [[Grid Search]] and [[Randomized Search]] are what the soft half consists of, and they carry the cost formulas and the two enumeration policies. [[Cross-Validation]] supplies the score candidates are ranked on, and [[Testing Set]] is what the invariant above exists to protect, since the reported quality of an automated search depends entirely on a number the search was not allowed to see.

[[Inductive Bias]] is where the search space lands as a concept: handing the architecture to a search does not remove the prior, it moves it into the grammar of what the search may express. [[Experiment Tracking]] is what a search of this size is unusable without, because the comparability conditions [[Model Selection]] states can only be checked against a record of what was actually tried, and a thousand-candidate search produces no such record by itself. [[Distributed Training]] is how the hard half's bill is paid at all, the 800-GPU figure above being a statement about parallel training runs rather than about one long one. [[Production Machine Learning]] is where the winner has to survive afterwards, and it is the reason a search that optimizes a single validation score can return a candidate that is not deployable: serving cost, latency and interpretability are not in $\widetilde{\mathcal{L}}$ unless they were put there.

### The tools, and what each is part of

Four products are worth naming, each with what it actually belongs to, because two of them are routinely filed under the framework they sit next to rather than the project that builds them.

- **auto-sklearn** is an independent project of Frank Hutter's group at the University of Freiburg, built *on top of* scikit-learn as a drop-in replacement for an estimator. It is not a scikit-learn utility and is not distributed with the library. Feurer, Klein, Eggensperger, Springenberg, Blum and Hutter, "Efficient and Robust Automated Machine Learning" (NeurIPS 2015), is the primary reference, with Auto-sklearn 2.0 in Feurer, Eggensperger, Falkner, Lindauer and Hutter (2020). Its last release is 0.15.0, February 2023, so anyone reaching for it today is reaching for something that has not moved in three years.
- **TPOT** searches whole scikit-learn pipelines, preprocessors and estimator together, by genetic programming; Olson and Moore (Proceedings of the Workshop on Automatic Machine Learning, PMLR 64, 2016). It was rewritten as a graph-based implementation and released as 1.x, latest v1.1.0 in July 2025.
- **Keras Tuner** is maintained by the Keras team as `keras-team/keras-tuner`, not by TensorFlow and not as a TensorFlow utility. Since version 1.4.0, September 2023, it targets multi-backend Keras, and from 1.4.6, November 2023, Keras 3, so it runs against TensorFlow, JAX or PyTorch as the backend; the current release is 1.4.8, November 2025. Its built-in strategies are random search, Hyperband and Bayesian optimization.
- **Ray Tune** is a library inside the Ray distributed-computing project and is framework-agnostic, wrapping PyTorch, TensorFlow, Keras or gradient-boosted trees alike; Liaw, Liang, Nishihara, Moritz, Gonzalez and Stoica, "Tune: A Research Platform for Distributed Model Selection and Training" (2018). What it supplies is the distribution and the schedulers, ASHA and population-based training among them, while integrating outside optimizers such as Optuna, HyperOpt, BOHB and Ax rather than owning one.

The tooling market itself, and which layer of an MLOps stack each of these belongs to, is owed to DMLS chapter 10, on infrastructure and tooling.
