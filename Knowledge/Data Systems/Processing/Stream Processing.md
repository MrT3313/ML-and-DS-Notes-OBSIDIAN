---
note_kind: concept
aliases:
  - stream processing
  - streaming data
  - stateful computation
up: "[[Data Systems]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Stream processing is computation over data in motion: the job reads records off a real-time transport as they are produced, rather than waiting for them to be written to a store and reading them back out afterwards. The transports themselves, and the publish-subscribe and message-queue shapes they come in, are [[Event-Driven Communication]]; stream processing is what a consumer of one does with what it reads.

The word "streaming" is overloaded in the machine learning domain, and not in this sense at all. [[Online Learning]] carries `streaming learning` as one of its names, and there the word describes how a fit is computed, one instance or one mini-batch at a time, with the parameters updated after each. That is a statement about an update rule and says nothing about where the instances came from or whether the supply of them ever ends: a model can be fitted that way from a file that has been sitting on a disk for a year, and a stream processing job can run for years without fitting anything. The two senses share the word and nothing else.

Processing this way is often low latency, because a record is handled as soon as it is generated instead of being written to a database and read out again, which takes a write and a read off the path entirely.

## Formal statement

**The unbounded-input contract.** Where a batch job is handed a fixed input set, a stream processing job is handed a sequence $e_1, e_2, e_3, \dots$ with no last element. Apache Kafka's own documentation states the abstraction the same way: a stream "represents an unbounded, continuously updating data set". Three consequences, all checkable:

- There is no moment at which the answer is final, because there is no moment at which the input is complete. A result is only ever a result *as of* a position in the sequence, so the honest form of an output is the pair (value, position), and a system that reports the value without the position has thrown away the only thing that makes it interpretable.
- The job's output is therefore not a function of an input set. It is a function of an input set *and* a cut point, which is why replaying the same stream from the same position is the only reproduction that means anything.
- The job must hold state, since the thing it is computing spans more records than the one in its hand.

**What the state buys, in arithmetic.** This is the whole case for the shape. Take a window of $W$ days recomputed on a daily cadence, with $r$ records arriving per day, and let $S$ be the state carried between runs.

$$\text{batch, per run} = Wr \qquad \text{incremental, per run} = r + |S|$$

A batch run reads the whole window every day, because it holds nothing from yesterday. The incremental run reads only the day of new records and joins the result against the stored state. The ratio is

$$\frac{Wr}{r + |S|} \;\approx\; W \quad \text{when } |S| \ll r$$

so the saving is a factor of the window length. The raw thirty day summary is $W = 30$: the batch job reads thirty days of data every day, and the incremental one reads one day and a small state object, for roughly a thirtieth of the reading.

The state stays small but it is not a single number. A window that rolls has to *evict* the day that falls out of it as well as absorb the day that enters, so the state holds $W$ per-day partial aggregates rather than one running total, and $|S|$ grows with $W$. It still satisfies $|S| \ll r$ by a wide margin, since $W$ partials are nothing against $Wr$ raw records, but the growth is in the right place to notice.

**What the factor of $W$ is paid for.** Two things, and both are the batch contract's guarantees being given up.

- The job must carry state between runs, so it has a lifetime and a memory. A batch job has neither, and can be killed and restarted with no consequence at all.
- Correcting a record is no longer a rerun. Under the batch contract a late or wrong record is fixed by editing the input and running the job again, and the answer is right by construction because the output is a function of the input. Here the bad record has already been folded into the state, so the correction is a *repair* of that state: Flink's answer is the checkpoint, which "marks a specific point in each of the input streams along with the corresponding state for each of the operators" so a job can be "resumed from a checkpoint while maintaining consistency (exactly-once processing semantics)", with manually triggered savepoints as the version you can replay deliberately from. That machinery is the cost the factor of $W$ buys, and it is why the rerun-from-the-start recovery of [[Batch Processing]] is a real guarantee rather than a limitation.

**What does not decompose.** The arithmetic assumes the aggregate can be maintained by adding the new day and subtracting the expiring one. Counts, sums, and anything built from them (a mean from a sum and a count) qualify. An exact median or an exact distinct count does not: the incremental version of either needs the window's records themselves or an approximation with its own error bound, and there the factor of $W$ is not available at full accuracy.

**On the efficiency argument.** Stream processing is commonly believed to be less efficient than batch, and the belief is not safe. Checked against the primary documentation rather than repeated: Apache Flink states that applications "are parallelized into possibly thousands of tasks that are distributed and concurrently executed in a cluster", reports deployments "maintaining multiple terabytes of state", "running on thousands of cores", and "processing multiple trillions of events per day", and keeps task state "in memory or, if the state size exceeds the available memory, in access-efficient on-disk data structures" so that state access is local. Kafka Streams derives its parallelism from the partitions of its input topics, assigning each task a fixed set of them, and keeps per-key state in local state stores made fault tolerant through changelog topics in Kafka itself.

What that supports is the scalability half: these are distributed, parallel, fault-tolerant systems operating at large scale, and the state that makes the incremental computation possible is held locally rather than fetched, which is the reason it is fast. What it does not support is a general claim that stream processing is more efficient than batch, and neither project's documentation makes one. It also names a real ceiling, visible in the Kafka Streams model: parallelism is bounded by how the input is partitioned, so a stream whose records all carry one key does not parallelize no matter how many machines are available.

**The cadence is a dial, not a switch.** Stream processing can itself be done in batches, on intervals of minutes where a batch pipeline runs on intervals of days, and a run can be kicked off whenever a need arises rather than only on the schedule. The tidy dichotomy is really a spectrum of cadence: as the interval shortens, the per-run input shrinks toward a single record and the incremental arithmetic above becomes the only sensible way to compute the window. What genuinely differs at the two ends is not the machinery but whether the job is promised an input that ends.

[[DDIA]] chapter 12 is titled Stream Processing and is unread. It carries what this note treats as a single word, "position": event time against processing time, ordering, windowing, watermarks and what a system does with a record that arrives after its window has been reported.

## Where it is used

[[Batch Processing]] is the other half of the axis, and it carries the property-by-property comparison of the two rather than the comparison being written twice. [[Event-Driven Communication]] is the layer underneath, supplying the transports a stream processing job reads from and the event vocabulary this note uses without defining. [[Online Analytical Processing]] is a consumer: the product-analytics systems described there ingest from a stream and make a record queryable as it arrives, which is what separates them from a warehouse fed by a scheduled pipeline.

[[Online Learning]] is the note the word "streaming" collides with, kept linked so a reader who arrived at the wrong one leaves immediately. [[Feature]] is where the machine learning reading of this contrast lives: which inputs of a model are computed from data at rest and which from the current state of the system is a question about features, and it is answered there rather than here.
