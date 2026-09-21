---
note_kind: concept
aliases:
  - business objective
  - business objectives
  - business metric
  - business metrics
up: "[[Machine Learning Systems Design]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

A business objective is the quantity the business is actually judged on, as distinct from the quantity a model optimizes and the quantity a model is scored by. For a machine learning project to succeed inside a business it is crucial to tie the performance of the system to overall business performance, and that tie is a claim about how those quantities relate rather than something the project gets for free.

There are three levels here and the vault must not flatten them. [[Performance Measure]] already holds the first two apart: the quantity optimized during training, usually a differentiable [[Cost Function]] chosen partly because a fitting procedure can move it, and the quantity reported to judge the trained model, computed on held-out data after fitting has stopped. The business metric is neither. It is not differentiable, it is usually not measurable offline at all, since it is a property of what people did after they were served, and it is what the decision is finally made on.

The second claim is the sharp one: many companies run experiments and choose the model that leads to better business metrics regardless of whether that model has better machine learning metrics. What follows from that is a point about evidence. The relation between the two is an empirical claim about one particular system, not an assumption that holds because both numbers are called performance, so it has to be tested rather than presumed. [[Production Machine Learning]] already records the production readiness test that says exactly this, that the offline proxy being optimized must be known to correlate with the online impact the system is judged by.

### Comparing two variants on live traffic

A/B testing is a controlled online experiment that splits live traffic between two variants and compares them on a metric fixed in advance. The mechanics, how traffic is split, how long a test has to run, and how a result is read without fooling yourself, are the subject of DMLS chapter 9, *Continual Learning and Test in Production*, and are not settled here.

### Whose interest the metric serves

One position on what the business metric ought to be has a name and a source. Milton Friedman argued that a business has one purpose and that it is profit, in "A Friedman Doctrine: The Social Responsibility of Business Is to Increase Its Profits", *New York Times Magazine*, 13 September 1970. His own sentence is that there is one and only one social responsibility of business, and he names it as:

> to use its resources and engage in activities designed to increase its profits so long as it stays within the rules of the game, which is to say, engages in open and free competition without deception or fraud.
> ~ Friedman, *A Friedman Doctrine: The Social Responsibility of Business Is to Increase Its Profits*, New York Times Magazine, 1970

"Maximize the profits for shareholders" is a paraphrase of that and not his wording; he writes "increase its profits", and of the executive that he is the agent of the owners.

This is a normative position from a 1970 newspaper essay, and it is contested rather than settled. Freeman's stakeholder theory (*Strategic Management: A Stakeholder Approach*, Pitman, 1984) holds that a firm is to be managed for the parties who affect it and are affected by it rather than for the owners alone. The Business Roundtable, an association of chief executives of large American companies, published a "Statement on the Purpose of a Corporation" on 19 August 2019, signed by 181 chief executives, committing its signatories to deliver value to customers, employees, suppliers and communities as well as shareholders, and superseding the shareholder primacy language its own statements had carried since 1997. So the position is carried here as one named claim with its source attached, and never as a statement of what businesses are for.

It belongs in this note because the choice is unavoidable rather than optional. Which quantity counts as the business objective is a choice about whose interest the system serves, so a system that optimizes a business metric has already answered that question whether or not anybody noticed.

## Formal statement

Write $M$ for the business metric and $\hat{P}$ for the offline performance measure, the held-out estimate [[Performance Measure]] defines. The contract has three clauses and one unproven assumption.

The model is fitted by minimizing a proxy of $\hat{P}$, usually a differentiable [[Cost Function]]. The trained model is scored by $\hat{P}$, computed offline on data that does not move. The decision is made on $M$, computed online on traffic the model has already been allowed to serve.

The assumption joining them is order preservation. For two candidates $A$ and $B$,

$$\hat{P}(A) > \hat{P}(B) \implies M(A) > M(B)$$

That is what "tie system performance to business performance" asserts, and it is neither guaranteed nor derivable from anything in the fitting procedure. It is a property of the particular system and the particular pair of metrics, and it is established, when it is established at all, by an experiment run online.

The failure follows immediately and is the whole reason the experiment is run. When the implication does not hold, every offline number can improve while $M$ fails to move or moves the wrong way, and nothing offline detects it, because every offline measurement is a function of the same held-out data $\hat{P}$ is computed from and none of them observes $M$. An offline pipeline reporting green is therefore consistent with a business metric getting worse, which is a checkable prediction and the one that justifies the cost of testing in production.

## Where it is used

[[Performance Measure]] is the level below this one, and the distinction between the two is what this note exists to keep. [[Machine Learning Systems Design]] is the activity whose first step is fixing this quantity, before there is a model to measure anything on. [[Machine Learning Project Lifecycle]] is where the loop closes: its step 1 sets this and its step 6 measures against it, which is what makes the sequence a cycle rather than a pipeline. [[Production Machine Learning]] is where $M$ becomes measurable at all, since it needs live traffic. [[Machine Learning Applicability]] asks this same question before anything is built, four of its nine conditions being about whether the investment returns what it costs. [[Decoupling Objectives]] is what happens when there is more than one of these at once, and the weights it argues over are a judgement about this quantity.
