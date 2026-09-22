---
note_kind: concept
aliases:
  - batch processing
  - batch job
  - batch jobs
  - offline processing
up: "[[Data Systems]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Batch processing is a bounded computation over data at rest. Once a record has arrived in a [[Storage Engine]] it is historical: it sits there unchanging, waiting to be read, and work over historical data is organized into *batch jobs*, runs that are handed a fixed set of stored records, read all of them, and finish. The input is bounded and known before the run starts, and everything else about the shape follows from that one property.

The word is overloaded in the machine learning domain, and not in this sense at all. A *batch* in [[Mini-Batch Gradient Descent]] is the set of training instances one parameter update is computed from, an argument to an update rule, and [[Batch Learning]] is the regime that sets that argument to the whole training set and fits once. Neither says anything about where the data sits, whether it is at rest, or whether the input is fixed before anything runs, and a batch job runs whether or not a model is ever fitted on what it produces. The two senses share the word and nothing else.

## VS

The axis is whether the input ends. A batch job is given a set of records that is complete before it starts, so it can read all of them and stop. [[Stream Processing]] is given a sequence that does not end, so it never stops and never has a final answer, only an answer as of a position in the sequence. Every row below follows from that.

| Property | Batch processing | Stream processing |
| --- | --- | --- |
| Input | bounded, fixed before the run starts | unbounded, no last record |
| Where the data is read from | a [[Storage Engine]] the record has already landed in | a real-time transport, as the record is produced |
| What starts the work | a schedule, or a request to kick it off | the arrival of a record |
| Typical cadence | hours to days | seconds to minutes |
| Status of a result | final for that input | as of a position in the stream |
| Recovery from a failure | rerun from the start, same answer | repair the carried state, then resume |
| State between runs | none required | required |

The cadence band is an order-of-magnitude comparison, taken from the contrast of minutes against days in [[DMLS Ch03 Data Engineering Fundamentals|DMLS chapter 3]], and it is not a threshold. A job does not become a stream processor by being scheduled every five minutes, and the boundary is soft in exactly that direction, which [[Stream Processing]] sets out.

That the split is in the input rather than in the machinery is what the engines themselves say. Apache Flink describes its Table API and SQL as "unified APIs for batch and stream processing", where "queries are executed with the same semantics on unbounded, real-time streams or bounded, recorded streams", and it calls a bounded stream a fixed-size data set it has dedicated operators for. One engine, two kinds of input.

## Formal statement

The contract is exact even though nothing here is numeric.

**What a batch job is given.** A run is handed an input set $D$ that is fixed at the moment the run begins. The job reads exactly $D$ and nothing that arrives after it starts. Its output is then a function of that input,

$$O = f(D)$$

with $f$ the job's logic. Two things follow immediately. The run is *repeatable*: executing it again on the same $D$ returns the same $O$. And a failed run needs no special handling, since rerunning it from the start cannot produce a different answer, which is why a batch job carries no state between runs and needs none.

The contract has a precondition, and it is the one that actually gets broken: $f$ must read nothing outside $D$. A job that calls the wall clock, that joins against a live table, or that appends to its own destination rather than replacing it, is not a function of its input, and rerunning it after a failure gives a different answer. That is checkable against a real pipeline by rerunning yesterday's job today and comparing.

**What the contract costs.** Take a job on a schedule of interval $T$ whose run takes $R$ to complete. A record that lands just after a run has started is not read until the next run, which begins at most $T$ later and finishes $R$ after that, so the age of the freshest record visible in the output lies in

$$[\,R,\; T + R\,]$$

Freshness is therefore bounded below by the cadence: no amount of making the job faster takes the staleness under $R$, and nothing takes the worst case under $T + R$ while $T$ stands.

**What does not follow.** Nothing in the contract says a batch job is slow. $T$ is a scheduling decision and $R$ is an engineering one, and a job with $T$ of one minute is as much a batch job as one with $T$ of one day. What the contract fixes is that there *is* a $T$, and that the freshest answer available is an answer about the world as it stood when the last run began.

**Why the cadence is usually long anyway.** Batch work is put on a slow schedule when the thing it computes changes slowly, which is the ordinary case rather than a compromise: recomputing at a cadence faster than the underlying quantity moves buys nothing and costs a full pass over $D$ each time. In a machine learning system that reasoning splits the inputs of a model into slow-moving and fast-moving ones, and [[Feature]] is where that split is written down.

[[DDIA]] chapter 11 is titled Batch Processing and is unread. It carries the mechanisms this note states only as a contract: how a bounded computation is distributed across machines, what the failure semantics of the distributed version actually are, and what the repeatability above costs to guarantee when the run spans a cluster.

## Where it is used

[[Stream Processing]] is the other half of the axis, and the comparison above is where the two are set against each other rather than repeated in both notes. [[Extract-Transform-Load]] is the standing example of a pipeline built to this shape: bulk scheduled work whose unit of output is a batch of rows rather than an answer to a question, with the repeatability precondition above showing up there as the demand that a backfill be idempotent. Its usual destination is a [[Data Warehouse]], and the workload the loaded rows then serve is [[Online Analytical Processing]], which is fed either by a pipeline of this shape or by a stream.

[[Storage Engine]] is where a record becomes historical, and so where it becomes a candidate for a batch job at all: under this shape the arrival of a record and the processing of it are two separate events, which is precisely what stream processing collapses into one. [[Batch Learning]] is the training regime run on a cadence for the same reason a batch job is, that the world moves slowly enough to allow it, though the "batch" in its name comes from elsewhere and is the collision the definition above separates.
