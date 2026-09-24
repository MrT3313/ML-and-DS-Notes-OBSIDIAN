---
note_kind: concept
aliases:
  - features
  - attribute
  - input variable
  - predictor variable
  - covariate
  - batch feature
  - batch features
  - streaming feature
  - streaming features
  - static feature
  - static features
  - dynamic feature
  - dynamic features
up: "[[Training Instance]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
  - "[[DMLS Ch05 Feature Engineering]]"
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---
## Definition

A [[Feature]] is one measurable property, characteristic, or attribute of an [[Training Instance|instance]]: a column of the data that the [[Model]] reads to make its prediction. 

> [!note]
> An "attribute" is the data type (mileage) while a "feature" is attribute plus value (mileage = 15,000), though the words are used interchangeably ~[[HOML Ch01 The Machine Learning Landscape|HOML Chapter 1]].

## Formal statement

Instance $i$ is the vector $\mathbf{x}^{(i)} \in \mathbb{R}^{n}$; feature $j$ is its component $x^{(i)}_j$, and the full feature matrix is $\mathbf{X} \in \mathbb{R}^{m \times n}$.

### Pipeline agreement

Once a model is deployed, feature $j$ is computed twice over its life: once by the path that builds training rows, $g^{\text{train}}_j$, and once by the path that answers a prediction request, $g^{\text{serve}}_j$. Both take an entity $e$ (the restaurant, the user, the order) and the moment $t$ the prediction is made for. The invariant a deployed system owes its model is

$$g^{\text{train}}_j(e, t) \;=\; g^{\text{serve}}_j(e, t) \qquad \text{for every entity } e,\ \text{every time } t,\ \text{every feature } j$$

with both sides reading only records stamped at or before $t$. The second clause is the no-time-machine requirement of [[Data Leakage]]: a training path that reads records later than $t$ breaks the equality in the one direction that flatters the offline score.

A violation is one cause of train-serving skew, and it is checkable: log the feature values the serving path actually produced, recompute them offline with the training path for the same $(e, t)$ pairs, and compare row by row. The distinctive thing about this cause is that it needs no change in the data at all. The underlying distribution can be identical in training and in production and the model still sees different inputs, because two pieces of code disagree about what the column means, which is why it is not the same failure as [[Data Mismatch]].

**Freshness.** How old the value is at the moment it is read depends on which kind of feature it is. A batch feature served from a job on interval $T$ with run time $R$ has age

$$a_{\text{batch}} \in [\,R,\; T + R\,]$$

which is the bound [[Batch Processing]] derives for any scheduled job and holds here unchanged. A streaming feature's age is bounded by the pipeline's own end-to-end delay $\delta$, from an event being produced to the updated value being readable by the prediction service,

$$a_{\text{stream}} \le \delta$$

and $\delta$ is set by the transport and the stream computation engine, not by any schedule. The two bounds are why the choice between kinds is a choice about how fast the quantity moves: when it changes much more slowly than $T + R$, the batch bound costs nothing.

## Where it is used

Features are the input side of every [[Model]]. Features that carry no signal are [[Irrelevant Features]]; reducing $n$ is [[Dimensionality Reduction]]; noisy or missing values are [[Poor-Quality Data]]. How much of what a fitted model does rests on one feature is [[Feature Importance]], and whether that contribution survives on data the model was not fitted on is [[Feature Generalization]]. In [[Supervised Learning]] the label is the one column that is not a feature.

### Batch and streaming features

Features split by where their inputs sit when they are computed. A *batch feature*, also called a *static feature*, is computed from historical data at rest by a scheduled job, which is [[Batch Processing]]. A *streaming feature*, also called a *dynamic feature*, is computed from data still in a real-time transport, as records arrive, which is [[Stream Processing]] over [[Event-Driven Communication]]. The two pairs of names are one distinction, not two.

A food delivery app estimating how long an order will take makes the split concrete. The restaurant's mean preparation time over past orders is a batch feature: it moves slowly, and recomputing it once a day loses nothing. The number of other orders the restaurant has taken in the last ten minutes, and the number of delivery people available right now, are streaming features: they describe the current state of the system, and yesterday's value of either is useless. One model reads all three, so the choice of kind is made per feature, never once for the whole model.

Which kinds a model can read follows from how its predictions are produced, which [[Batch and Online Prediction]] sets out. Batch prediction reads only batch features, since its predictions are computed ahead of any request from data already at rest. Online prediction can run on batch features alone, looked up at request time. Streaming prediction is online prediction that reads streaming features as well, alongside the batch ones.

A running system gets a streaming feature from a streaming pipeline, which has two parts. A real-time transport carries the events (a new order, a courier going on or off shift) as they are produced, and a stream computation engine reads them off it and keeps the aggregate current as state, a count over a ten-minute window here, writing the value where the prediction service can read it. At request time the service joins that value with the batch features for the same entity and passes the row to the model. That is what lets an online prediction reflect the last few minutes: the pipeline supplies inputs within $\delta$ of the events that caused them, and the model's own speed is the other half of the latency budget.

### One pipeline or two

[[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]] reads batch prediction as largely a legacy of the systems that were available: large-scale data processing was for years dominated by batch engines such as MapReduce (Dean and Ghemawat, OSDI 2004), which process large volumes of stored data efficiently on a schedule. That is Huyen's historical reading rather than a measured claim, and the dates are consistent with it. Spark, often named in the same breath, is batch only in its origins. It was introduced as a batch and interactive engine (Zaharia et al., "Resilient Distributed Datasets", NSDI 2012), gained a streaming model a year later in discretized streams, which runs a stream as a series of short batch jobs and is designed to compose with batch queries (Zaharia et al., SOSP 2013), and since Structured Streaming (experimental in Spark 2.0, July 2016; no longer experimental from Spark 2.2, July 2017; Armbrust et al., SIGMOD 2018) the same DataFrame or SQL query runs on stored or streaming input.

The trouble starts when a team that already has a batch pipeline wants streaming features for online prediction. The usual move is to build a second, streaming pipeline next to the first, and then the same feature exists as two implementations: the batch one computes it over history to build training rows, and the streaming one computes it live at serving time, often in a different engine and a different language. Every difference between the two, a window boundary drawn on a different side, a late event counted in one and dropped in the other, is the pipeline agreement invariant above broken, and it produces a model that is quietly wrong in production with nothing raising an error. Two pipelines doing one job is a common source of bugs in production machine learning for exactly this reason. Deploying a simple model early, which [[Model Selection]] recommends, is partly a way to find this disagreement before anything expensive depends on it.

The shape has a name in data systems. Marz's architecture, published as a blog post in 2011 and as the Lambda architecture in Marz and Warren, *Big Data* (Manning 2015), runs a batch layer that recomputes over all data and a speed layer that covers only what the batch layer has not reached yet, and merges the two at query time: the same function written twice by design. Kreps's objection ("Questioning the Lambda Architecture", O'Reilly Radar, July 2014) is the machine learning complaint in general form: "maintaining code that needs to produce the same result in two complex distributed systems is exactly as painful as it seems like it would be." His alternative, since called the Kappa architecture, keeps one stream processing path and handles reprocessing by replaying the retained log through a second instance of the job into a fresh output, which works as far back as the transport's retention window reaches.

[[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]] (2022) presents unifying the two as an active area of research. The sources support something firmer: it has standing engineering answers, and has had them for a decade. The Dataflow model (Akidau et al., VLDB 2015) argues for dropping the assumption that a dataset ever becomes complete and treating bounded data as a case of unbounded, so one program serves both. Flink's documentation, as of its 2.x releases in September 2026, describes its Table API as running a query on batch or streaming input without modification (the older wording is quoted in [[Batch Processing]]), and Structured Streaming does the same inside Spark. On the machine learning side, Zinkevich's "Rules of Machine Learning" (Google) names a discrepancy between how the training and serving pipelines handle data as a cause of training-serving skew, and gives two remedies: reuse code between the training and serving pipelines wherever possible (rule 32), and log the features actually used at serving time so they can be used for training (rule 29), which makes the two sides equal by construction rather than by care. What remains hard is engineering rather than an open question: backfilling training rows over months of history through logic written for a live stream, with each value computed as of its own $t$. The store that serves both kinds of feature behind one interface, the feature store, is [[DMLS]] chapter 10's subject.
