---
note_kind: concept
aliases:
  - batch prediction
  - batch predictions
  - online prediction
  - online predictions
  - on-demand prediction
  - synchronous prediction
  - asynchronous prediction
  - streaming prediction
  - batch inference
  - online inference
  - real-time prediction
  - real-time inference
  - batch serving
  - online serving
  - prediction mode
  - prediction modes
up: "[[Model Inference]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## Definition

Batch prediction computes predictions before anyone has asked for them, on a schedule or when something triggers the job, and stores them, in a SQL table or an in-memory store, where a request later looks up the stored answer. Online prediction, also called on-demand prediction, computes a prediction only when the request for it arrives and returns it straight away.

Two further names ride on these. *Synchronous prediction* is online prediction answered over a request the caller waits on, ex an HTTP call to a prediction service, which is [[Request-Driven Communication]], one of the three [[Modes of Data Flow]]. *Asynchronous prediction* is [[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]]'s other name for batch prediction, in the sense that the prediction is computed at a different time from the request. The vendor term is not the same thing: Amazon SageMaker's Asynchronous Inference (checked September 2026) is an endpoint that queues each incoming request and processes it after the call returns, with results written to S3; nothing is computed ahead of a request, so it is online prediction with the waiting removed from the caller, not batch prediction.

*Streaming prediction* is not a third mode beside the two. The modes sit on two independent axes: **when** the prediction is computed, ahead of the request or on it, and **what it reads**, only batch features or at least one streaming feature ([[Feature]] carries both kinds). Batch prediction runs before the request exists, so it cannot read a feature describing the state of the system at request time, and one cell is empty:

| | batch features only | batch and streaming features |
|---|---|---|
| computed ahead of the request | batch prediction | impossible |
| computed on the request | online prediction with batch features, ex precomputed embeddings looked up at request time | streaming prediction |

The three occupied cells are the three modes usually listed. The middle one is still online prediction; what it lacks is fresh inputs, not freshness of computation.

This axis is not the training regime. Batch and online prediction say when a *fitted* model is asked; [[Batch Learning]] and [[Online Learning]] say when the model is *fitted*. The two are independent, so a system can sit in any of the four combinations, ex a model trained in batch and served online.

## VS

**When batch pays.** Batch prediction is the right choice when many predictions are wanted and nobody needs them the moment they are asked for: the whole job runs as one bulk call, which [[Model Inference]] shows is the cheapest way to compute a prediction per row, and the request path shrinks to a key lookup. [[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]] works it with recommendations: a streaming service could recompute every user's recommendations every four hours and serve the stored list when the user logs in. That is an illustration of the shape, not documented practice at any company; Huyen's own "Real-time machine learning: challenges and solutions" (January 2022) names Netflix as a batch-prediction example circa 2021 and adds that it was moving its predictions online.

**What batch costs.** Two things, both structural rather than engineering.

- *Staleness.* A stored prediction describes the user as they were when the job last read the data, so a change of preference is invisible until the next run. The formal statement bounds how stale.
- *Guessing the requests.* The job has to know in advance which rows will be asked for, so it computes a prediction for every candidate, including the ones nobody requests before the next run replaces them. A query that could not have been enumerated, a new search string, a user who signed up an hour ago, has no stored answer at all.

**What online needs.** Two things, and the second is the one that is usually short.

- A near-real-time pipeline that takes the incoming request, computes any streaming features it needs, feeds them to the model and returns the answer. The features come off a real-time transport ([[Event-Driven Communication]]) and are computed by a stream engine ([[Stream Processing]]); [[Feature]] carries how batch and streaming features are produced and kept consistent.
- A model that answers fast enough for whoever is waiting. For an interactive consumer product that means a budget in milliseconds, and the budget is for the whole response, not the model: Google's RAIL model sets 100 ms from user input to visible response as the threshold under which a reaction feels immediate, which descends from the 0.1 second limit Nielsen gives for a system to feel instantaneous (Nielsen, "Response Times: The 3 Important Limits", 1993, after Miller 1968 and Card et al. 1991). What is left for the model is that total minus everything else in the [[Model Inference]] latency sum, and making the model fit that remainder is what [[Model Compression]] is for.

[[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]] observes that engineers from an academic background find online serving the more natural default. That is the author's observation about practitioners rather than a measured fact.

## Formal statement

**Staleness of a batch prediction.** Let the job start every $\Delta$ and take $R$ to finish, reading its inputs as they stand when it starts. A run that starts at $s$ publishes at $s + R$, and its predictions are served until the next run publishes at $s + \Delta + R$. A request at time $t$ in that window receives a prediction computed from data as of $s$, so its age is

$$a = t - s \in [\,R,\; \Delta + R\,)$$

which is the freshness interval $[R, T + R]$ of [[Batch Processing]] applied to served predictions, with $T = \Delta$. When requests arrive uniformly over the serving window, $a$ is uniform on that interval and

$$\mathbb{E}[a] = R + \frac{\Delta}{2} \qquad a_{\max} = \Delta + R$$

The four-hour example with a one-hour job serves predictions three hours old on average and up to five. Shortening $\Delta$ lowers both, and $R$ is a floor neither can go under.

**Wasted work.** Let each run compute $N$ predictions, of which a fraction $f$ are requested before the next run replaces them. The run wastes

$$W = (1 - f)\,N$$

predictions. Let $c_b$ be the cost of one prediction computed in bulk and $c_o$ the cost of one computed on request, with $c_b < c_o$ because of the bulk advantage. If each requested row is asked for once, the two modes cost $c_b N$ and $c_o f N$ per interval, and batch is cheaper exactly when

$$f > \frac{c_b}{c_o}$$

So batch wins on compute when enough of what it precomputes gets used, and the cheaper the bulk call is relative to the online one, the lower that bar. The inequality prices compute only: it says nothing about staleness, which batch pays whatever $f$ is, and a row requested $k$ times costs online $k\,c_o$, which moves the bar in batch's favour.

**Online.** Nothing is precomputed, so $a = 0$ for the computation and $W = 0$, and what is paid instead is the end-to-end latency of [[Model Inference]] on every request. The mode is feasible exactly when

$$L_{(q)} \le \text{budget}$$

at the percentile $q$ the product cares about, with the budget set by whoever is waiting, 100 ms in total for an interaction meant to feel immediate.

## Where it is used

[[Production Machine Learning]] is the setting, and its paragraph on batch serving across the major cloud platforms and the large-model providers is the evidence that batch is a first-class mode rather than a legacy one; it is not restated here. [[Model Inference]] is the operation both modes schedule, and it carries the latency sum and the bulk-against-atomic throughput relation the formal statement leans on.

[[Batch Learning]] and [[Online Learning]] are the other axis, when a model is fitted rather than when it is asked, and the words "batch" and "online" mean something different in each. [[Feature]] is where batch and streaming features are defined, and so where the second axis of the table above is decided per input. [[Batch Processing]] supplies the freshness interval that the staleness bound applies, and a batch prediction job is a batch job in its sense, with the fitted model as its $f$. [[Stream Processing]] and [[Event-Driven Communication]] are the machinery streaming prediction depends on and batch prediction does without.
