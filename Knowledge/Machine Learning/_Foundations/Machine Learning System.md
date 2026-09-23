---
note_kind: concept
aliases:
  - machine learning system
  - machine learning systems
  - ML system
  - ML systems
  - machine learning application
  - machine learning applications
  - ML application
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

A **machine learning system** is not a model. It is the whole thing a model sits inside, and it comes in three parts: the code that runs, the data that code was fitted to, and the artifacts that come out of putting those two together. The distinction is worth drawing because the system's behaviour is settled by all three at once, so reading the code tells you only a part of what the thing will do.

## VS

Traditional software, where the deliverable is the program and the data is something that shows up while it runs.

|                                                  | Traditional software                                 | Machine learning system                                                                                            |
| ------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Deployed behaviour is a function of              | the program text                                     | the code, the training data, and the artifact fitted from the two                                                  |
| What is kept under version control               | the source                                           | the source, the training data, and the fitted artifact, each pinned on its own                                     |
| What a test asserts                              | a stated output for a stated input, an equality      | a score over a held-out sample, a threshold that passes or fails statistically                                     |
| How a regression gets introduced                 | somebody edited a line                               | somebody changed the training data, or refitted on newer data, with no line edited                                 |
| What reproducing a reported behaviour takes      | the commit it was seen on                            | that commit, the training data as it stood, and the artifact, since refitting is not guaranteed to land in the same place |
| How a failure presents itself                    | something breaks and the service says so             | nothing breaks, every component keeps answering, and the answers get worse ([[Model Rot]])                         |

The axis the table turns on is usually stated as software engineering keeping code and data apart. That wants stating carefully, because as a principle it is not quite what the primary sources say. Dijkstra's separation of concerns is about where attention goes, not about which artifacts are kept apart:

> We know that a program must be correct and we can study it from that viewpoint only; we also know that it should be efficient and we can study its efficiency on another day, so to speak. [...] It is what I sometimes have called 'the separation of concerns', which, even if not perfectly possible, is yet the only available technique for effective ordering of one's thoughts, that I know of.
> ~ Dijkstra, EWD447, *On the Role of Scientific Thought*, 1974

And Parnas, setting out the criterion for cutting a system into modules, puts a data structure and the code that touches it deliberately in the same place:

> A data structure, its internal linkings, accessing procedures and modifying procedures are part of a single module. They are not shared by many modules as is conventionally done.
> ~ Parnas, *On the Criteria To Be Used in Decomposing Systems into Modules*, 1972

So the separation that actually does the work is narrower, and it is about the deliverable. In traditional software the data is input: it arrives while the program runs, it is not part of what ships, and it is not what a version identifier or a test case names. In a machine learning system the training data is inside the deliverable, carried there by the fitted artifact, and every row of the table above follows from that one fact. Modularity is a design goal there and unreachable here, because the artifact has no definition that does not mention the data.

## Formal statement

Write $C$ for the code, $D$ for the training data, and $A$ for the artifact fitted from the two. The deployed behaviour $B$ of a machine learning system is a function of the triple jointly:

$$B = f(C, D, A)$$

Pinning $C$ alone therefore does not determine $B$, and reproducing an observed behaviour means pinning all three. The claim is falsifiable in the ordinary way: it predicts that two deployments running byte-identical code can behave differently, which is exactly what happens when one of them was fitted on a different $D$.

$A$ is itself a function of the other two,

$$A = \text{fit}(C, D)$$

which makes the triple redundant in principle and not in practice. Refitting is expensive, and it is not always deterministic: unseeded initialization, shuffling, and the order in which parallel workers reduce floating point sums can each land a second fit somewhere slightly different from the first. So $A$ is stored and shipped rather than recomputed on demand, and it is identified in its own right.

Versioning the data is the half of this that ordinary software has no occasion for, and it is load-bearing here for one reason: $A$ is a function of $D$, so a statement about $A$ that does not say which $D$ is not a statement about anything. Two training sets of the same size are not interchangeable, which is why the identifier has to resolve to the exact rows fitted on. A row count, a byte size or a date is not such an identifier.

What a data version pins is not settled vocabulary, and named systems differ on it. DVC records a content hash of the tracked file or directory in a small metafile that goes into git while the bytes go to a cache or a remote, and its lock file extends the same treatment to a pipeline stage by hashing that stage's dependencies and outputs alongside the command that produced them. Delta Lake takes the other route and numbers commits to a table, so a query asks for a version number or a timestamp rather than for a hash. Either satisfies the requirement, and the requirement is the fixed part: one identifier resolving to one state of the data.

## Where it is used

[[Model]] is the third part of the triple under its own name, the artifact fitted from code and data, and this note is the system it is one part of rather than the whole of. [[Training Set]] is the data part, the $D$ every claim above is conditioned on. [[MLOps]] is the practice of operating one of these once it ships, and it is where versioning the data as well as the code actually has to be carried out rather than merely asserted. [[Experiment Tracking]] is that pinning done to one training run rather than to a deployment, naming the arguments $\text{fit}$ reads so that each of the three is recorded separately, and a run is repeatable only when all three are. [[DevOps]] is the same practice for a system whose behaviour is written by hand instead of learned, and it already states what MLOps adds that it lacks. [[Model Rot]] is the failure mode that follows from the data part: nothing in the code changes and the system gets worse anyway.

[[Machine Learning Applicability]] is the set of conditions a problem has to meet before building one of these is the right move at all, and [[Production Machine Learning]] is what one is asked for once it is serving traffic rather than being built. [[Data-Intensive Application]] is the neighbouring domain's answer to the same kind of question, what sort of system this is, and the two classifications cut on different axes: that one sorts systems by where the difficulty lives, this one by what the behaviour is made of.
