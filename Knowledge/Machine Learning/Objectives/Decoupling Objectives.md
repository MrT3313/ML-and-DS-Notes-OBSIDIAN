---
note_kind: concept
aliases:
  - decoupling objectives
  - decoupled objectives
  - objective decoupling
  - multiple objectives
up: "[[Performance Measure]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

Decoupling objectives is the choice a system faces once it has more than one objective: combine them into a single scalar before fitting, or fit one model per objective and combine their outputs at serving time.

The singular case is already split three ways here. A [[Performance Measure]] is the quantity, and its two sign conventions are a [[Cost Function]], which is minimized, and a [[Utility Function]], which is maximized. An objective function guides learning by minimizing the loss, or equivalently maximizing the gain, caused by a model's predictions; those are one instruction written with opposite signs, since $U = -J$ up to a constant, and optimizing is the genus of the two rather than a third option alongside them.

Having several of these at once is a different problem rather than a harder version of the same one. Framing a machine learning problem gets hard when there are several objective functions to optimize at once, because nothing in the problem now says how a candidate that wins on one and loses on another should be ranked, and something has to supply that before anything can be fitted at all.

## Formal statement

Let there be $n$ objectives with per-objective losses $\mathcal{L}_1, \dots, \mathcal{L}_n$, and let $\boldsymbol\alpha = (\alpha_1, \dots, \alpha_n)$, with $\alpha_j \geq 0$, be the weights saying how much each one counts.

**Combined.** Scalarize before fitting. The training objective is

$$\mathcal{L} = \sum_{j=1}^{n} \alpha_j \mathcal{L}_j$$

and the model that comes out of it is

$$h^{*}_{\boldsymbol\alpha} = \arg\min_{h} \sum_{j=1}^{n} \alpha_j \mathcal{L}_j(h)$$

which carries $\boldsymbol\alpha$ in its subscript because it genuinely depends on it. Changing any weight changes the objective, a changed objective changes the $\arg\min$, and a changed $\arg\min$ is a different model, which means a refit. Exploring $k$ distinct weight settings therefore costs $k$ fits.

**Decoupled.** Fit one model per objective,

$$h^{*}_{j} = \arg\min_{h} \mathcal{L}_j(h), \qquad j = 1, \dots, n$$

which is $n$ fits, done once, and combine the outputs at serving time:

$$s(\mathbf{x}) = \sum_{j=1}^{n} \alpha_j h^{*}_{j}(\mathbf{x})$$

Here $\boldsymbol\alpha$ enters after the fitting rather than before it. No $h^{*}_{j}$ mentions it, so changing it changes a scoring rule and nothing that was fitted. Exploring $k$ weight settings still costs $n$ fits.

**The comparison.** Combined costs $k$ fits and decoupled costs $n$ fits, so decoupling is the cheaper arrangement exactly when

$$k > n$$

Stating it as a crossover is what makes it checkable, and it rules something out: two objectives with a weighting nobody intends to revisit is $k = 1$ against $n = 2$, and there combining is cheaper, so "decouple by default" is not a claim this arithmetic supports on its own. The counting assumes one fit costs about the same in either arrangement, and that is the assumption to check first when the answer comes out wrong. A per-objective model can be smaller and cheaper to fit than the joint one, which moves the crossover further in decoupling's favour, while $n$ models in production cost storage, monitoring and a serving path each, which the single number $n$ does not capture.

The maintenance half is not arithmetic and runs the same way. The $n$ models can be maintained on separate schedules, because each depends on only its own objective and on nothing the others depend on, whereas one combined model is retrained as a unit whenever any single one of its objectives goes stale. [[Continual Learning]] is what a retraining schedule is and what fixes its cadence.

The weighting $\boldsymbol\alpha$ is a preference supplied from outside the optimization rather than derived from it: nothing in $\mathcal{L}_1, \dots, \mathcal{L}_n$ implies a value for any $\alpha_j$. The order theory behind that, Pareto dominance, the frontier, and why two candidates with every number measured can still be incomparable, is stated in [[Production Machine Learning]] and is not repeated here.

### What a weighted sum can reach

The combined arrangement has a second cost, and it is a limit on expressiveness rather than on price. Minimizing a weighted sum with non-negative weights returns a point on the convex hull of the Pareto frontier. Every solution of the weighted problem is Pareto optimal, but the converse, that every Pareto optimal solution is returned by some weighting, holds only when the problem is convex (Miettinen, *Nonlinear Multiobjective Optimization*, 1999). Where the frontier has a non-convex region, the trade-offs in that region are Pareto optimal and no choice of $\boldsymbol\alpha$ ever returns one of them; Das and Dennis give the geometric argument for why ("A closer look at drawbacks of minimizing weighted sums of objectives for Pareto set generation in multicriteria optimization problems", *Structural Optimization* 14, 1997, 63 to 69).

So the combined arrangement is not only more expensive to retune, it cannot express some of the trade-offs available to the system. The result bites where the weighted sum is what gets minimized, which is the combined case. The decoupled case fits nothing jointly, so it is not searching that frontier at all; what linearity bounds there is the family of rankings $\boldsymbol\alpha$ can induce over $n$ models already fixed, which is a smaller claim and a different one.

## Where it is used

[[Production Machine Learning]] is the setting that creates the problem, several parties each wanting something different out of one model, and it is the note that owns the order theory above. [[Performance Measure]] is the singular case this generalizes, and [[Cost Function]] is what the combined arrangement's single scalar is once the weights are fixed. [[Machine Learning Systems Design]] is the activity that decides how many objectives a system has and how they combine, which is this decision made before there is anything to fit. [[Continual Learning]] is where the separate maintenance schedules are actually run. [[Business Objective]] is usually what the several objectives are proxies for, which is why the weighting is a business decision wearing a coefficient.

The worked case is a ranking system: two models, each optimizing one objective, their outputs combined, and posts ranked by the combined score. It is the $n = 2$ instance of $s(\mathbf{x})$ above, and it is where the language of a combined score comes from.
