---
note_kind: concept
aliases:
  - machine learning project lifecycle
  - ML project lifecycle
  - machine learning lifecycle
  - ML lifecycle
  - machine learning project life cycle
  - four phases of ML development
  - 4 phases of ML development
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Developing a machine learning model is an iterative process and, in most cases, a never-ending one. Concretely, the work has six named phases and the sixth returns to the first, so the process has no terminal state: a project is not finished, it is between iterations.

### The six phases

```mermaid
---
config:
  themeCSS: |
    .edge-pattern-solid[id*="-P6-P2"], .edge-pattern-solid[id*="-P2-P6"],
    .edge-pattern-solid[id*="-P5-P3"], .edge-pattern-solid[id*="-P3-P5"],
    .edge-pattern-solid[id*="-P6-P3"], .edge-pattern-solid[id*="-P3-P6"],
    .edge-pattern-solid[id*="-P5-P2"], .edge-pattern-solid[id*="-P2-P5"] {
      stroke-dasharray: 6 4;
    }
---
block-beta
  columns 3
  space P1["1. Project<br>scoping"] space
  P6["6. Business<br>analysis"] space P2["2. Data<br>engineering"]
  space:3
  P5["5. Monitoring and<br>continual learning"] space P3["3. ML model<br>development"]
  space P4["4. Deployment"] space

  P1 --> P2
  P2 --> P3
  P3 --> P4
  P4 --> P5
  P5 --> P6
  P6 --> P1

  P6 --> P2
  P2 --> P6
  P5 --> P3
  P3 --> P5
  P6 --> P3
  P3 --> P6
  P5 --> P2
  P2 --> P5
```

1. **Project scoping.** Goals, objectives and constraints are set, stakeholders are identified, and resources are estimated and allocated.
2. **Data engineering.** Raw data arriving from different sources and in different formats is handled, and a training set is curated out of it by sampling and by generating labels. The phase borrows its name from [[Data Engineering]], which is a note in the neighbouring data domain about who does that work and what infrastructure they own, a standing role held whether or not a model is ever fitted on anything it moves, while what is named here is one phase of one project.
3. **Machine learning model development.** Features are extracted and initial models are developed on the engineered features. The extraction is [[Feature Engineering]], which is the map applied to every instance rather than the phase the map gets written in.
4. **Deployment.** The model is made accessible to its users.
5. **Monitoring and continual learning.** The model is monitored for performance decay, and maintained so that it stays adaptive to changing environments and changing requirements. The decay is [[Model Rot]] under another of its names, and the maintenance is [[Continual Learning]].
6. **Business analysis.** Model performance is evaluated against business goals and analysed to generate business insight.

## Formal statement

Read the six phases as the nodes of a directed graph, numbered as above. A picture cannot be checked against a claim and a stated adjacency can, so the adjacency is what follows.

Six forward edges form a single cycle:

$$1 \rightarrow 2 \rightarrow 3 \rightarrow 4 \rightarrow 5 \rightarrow 6 \rightarrow 1$$

Four further connections run in both directions, and they are not arbitrary. Every one of them joins one of the two late phases, 5 monitoring and continual learning and 6 business analysis, to one of the two build phases, 2 data engineering and 3 model development. The four are 6 with 2, 6 with 3, 5 with 2, and 5 with 3, which is every pair in the product of those two sets:

$$\{5, 6\} \times \{2, 3\}$$

So the feedback in this process is not a general licence for any phase to send you back to any other. It is exactly the complete bipartite connection between the phases that learn something from a running system and the phases that build one, eight directed edges on top of the six forward ones, fourteen in all, and no back edge touches 4 deployment or 1 project scoping.

That is the falsifiable claim, and what it implies is worth stating separately. The phases that generate evidence are the late ones, and the phases that can act on evidence are the build ones. Deployment is therefore a pass-through, carrying the model from development to the place evidence starts arriving without ever being a destination, and project scoping is reached only by completing a full cycle, which is to say only through business analysis.

The never-ending claim is a property of the same graph. The cycle above passes through every node, so no node is terminal: from wherever a project currently is, there is a path onward, and there is no vertex with no outgoing edge. A project's state is therefore a position in the cycle rather than a percentage of completion, and the number of iterations run is not bounded by anything in the structure.

### The hierarchy of needs

Nothing at the learning level of an organization is reachable until the layers under it are in place, and there are four of them: collecting, moving and storing, exploring and transforming, and aggregating and labelling. That is the same ordering step 2 of the loop asserts, a training set curated out of raw data before step 3 has anything to fit. Three of those four layers are the subject of the neighbouring data domain rather than this one. [[Data Engineering]] is the role that owns the moving and the storing, [[Extract-Transform-Load]] is the machinery the moving and the transforming are usually made of, and [[Data Systems]] is the domain both of those sit in.

The ordering is Monica Rogati's, from "The AI Hierarchy of Needs", published on Hacker Noon in June 2017 as a blog post rather than as a reviewed paper. That is worth saying plainly, because a pyramid published that way is weak evidence for anything structural, and only the precedence claim above is taken from it.

### The four phases inside model development

Phase 3 of the cycle, machine learning model development, is not entered the same way twice. Four phases sit inside it, written $d_1$ to $d_4$ and numbered separately from the six nodes above because they subdivide node 3 rather than extend the sequence around it. They also attach to a problem type rather than to a team, so one organization sits at a different $d_i$ for every problem it is solving.

- **$d_1$, before machine learning.** Nothing is fitted. The solution is a non-learned heuristic, ex recommending the most popular item or ordering by recency, and one legitimate outcome of this phase is the finding that no learned model is needed at all.
- **$d_2$, simplest machine learning models.** The first fitted model for this problem, chosen as the simplest thing that could work rather than the strongest thing available. That choice is a judgement about candidates and belongs to [[Model Selection]], which carries the argument for making it.
- **$d_3$, optimizing simple models.** The same family pushed as far as it will go, through the objective function, hyperparameter search, features, more training data, and ensembles of simple learners.
- **$d_4$, complex models.** Entered once the simple family has run out, and carrying an obligation the earlier phases do not: measuring how fast the model decays, which is what gives the retraining in phase 5 a schedule.

$$d_1 \prec d_2 \prec d_3 \prec d_4$$

One claim makes that order checkable rather than a taste for simplicity: the best solution reached in $d_i$ is the floor that the first candidate of $d_{i+1}$ has to clear. Every transition is therefore licensed by a number the previous phase left behind, which is the role [[Baseline Model]] plays within a single fit raised one level, the thing to beat being the previous phase's best rather than a constant predictor. A phase is left when it stops paying, and [[Learning Curve]] is the instrument that reads that condition off a fitted candidate: curves flat, close together and high say the family is saturated and that the next move is $d_4$ rather than more data ([[Underfitting]]), while a gap still open at the largest training size says the work remaining is inside $d_3$ ([[Overfitting]]). [[Model Debugging]] arrives at the same first step from the other side, since a simple implementation is the one whose failures can be localized at all.

$d_1$ is the phase most often skipped, and two rules of thumb make it concrete, both of them from Huyen's earlier public booklet *Machine Learning Systems Design* rather than from the reading this note records, which is worth saying because each attaches a number to a judgement. On the value of not skipping it, Martin Zinkevich's handbook "Rules of Machine Learning: Best Practices for ML Engineering" gives the estimate that "if you think that machine learning will give you a 100% boost, then a heuristic will get you 50% of the way there". On when to leave it, a heuristic stack grown past about a hundred nested conditionals costs more to maintain than a fitted model would, and that is the signal to move to $d_2$.

## Where it is used

[[Machine Learning Systems Design]] is the activity whose object this is: the loop is the shape of the work, and the design is what fixes the requirements each turn of it has to satisfy. [[Machine Learning Applicability]] is the gate in front of step 1, checked before goals, stakeholders and resources are worth fixing at all. [[MLOps]] is the practice covering steps 4 and 5 and only those, a boundary drawn deliberately, which is why the loop crossing back into steps 2 and 3 is not that practice widening.

[[Model Rot]] is what step 5 monitors for, and [[Continual Learning]] is what step 5 does about it. [[Data Engineering]] is the role that owns the infrastructure step 2 runs on, in a domain whose subject stands whether or not a model is ever fitted. [[Business Objective]] is what step 1 fixes and what step 6 measures against, which is what makes the last edge back to the first phase a comparison rather than a restart. [[Production Machine Learning]] is the setting steps 4 through 6 take place in, and the reason there is a step 6 at all: a model that serves live traffic is judged by what it does to the business rather than by the score it was selected on.
