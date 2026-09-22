---
note_kind: concept
aliases:
  - event-driven
  - event driven
  - event-driven architecture
  - real-time transport
  - publish-subscribe
  - pubsub
  - pub/sub
  - message queue
  - message queues
  - message broker
  - event stream
up: "[[Distributed System]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

An event is a record that something happened, broadcast to a shared transport rather than addressed to a recipient: it reports a fact about the past, it asks for nothing, and the process that produces it does not wait for a reply. **Event-driven communication** is the mode of data flow built out of those broadcasts, in which a producer hands an event to a real-time transport and every process that has arranged to read from that transport can receive it.

The transport is a broker, a service whose whole job is to accept events, hold them and hand them on. Putting one in the middle is what buys everything in the contract below, and the broker is also what the arrangement now depends on.

## Formal statement

Stated against the contract in [[Request-Driven Communication]], the mode changes three clauses and keeps one.

- **The producer names a destination, not a consumer.** It publishes to a topic or a queue, which is a name the broker owns, not the address of a process. Adding a consumer changes nothing at the producer.
- **The consumer need not be running when the event is produced.** The broker accepts the event and holds it, so producing and consuming are separated in time as well as in space.
- **The producer learns that the broker accepted the event, and nothing further.** Whether anyone read it, whether the reader succeeded, and how long any of that took are all outside what the publish call returns.
- **Something in the middle must still be running**, and it is now the broker rather than the consumer. This is why brokers replicate their data across machines: the mode concentrates the availability requirement into one component instead of spreading it across every consumer.

The consequence is that availability no longer composes the way it does down a chain of synchronous calls. A consumer that is down does not fail the producer, it falls behind, and it catches up when it returns. The price is paid somewhere less convenient. Failure moves off the call site, where the caller's own code is standing there waiting for it, to a place nobody is watching: the publish succeeded, so the only symptoms of a broken consumer are an output that never arrives and a lag measurement that somebody has to have thought to monitor.

The holding is bounded rather than permanent, and that bound is the fine print on the second clause. A broker retains events for a configured window $R$ and discards what is older, so a consumer that resumes after an outage of duration $t$ loses nothing when $t \le R$, and permanently loses the events published during the first $t - R$ of its outage when $t > R$. That inequality is the operational content of a retention setting, and it is what turns "the consumer need not be running" from an unconditional promise into a bounded one.

### The two shapes, and how much of the split survives

The two shapes this mode is usually presented in differ on exactly one thing, which is whether an event has an intended recipient.

- **Publish-subscribe.** Any service can publish to a topic in the transport, and any service that subscribes to that topic can read every event in it. A service that produces data does not know and does not care which services consume it.
- **Message queue.** The event has intended consumers, and the queue is responsible for getting each message to the right one, delivering it to a consumer rather than to all of them.

That split is accurate as a description of two delivery semantics and misleading as a classification of systems, because the brokers people actually run offer both. Kafka is the clean case. Consumers sharing a `group.id` form a consumer group, and Kafka's documentation for its consumer client states that Kafka "will deliver each message in the subscribed topics to one process in each consumer group", achieved by assigning each partition to exactly one consumer in the group. Put every consumer in a single group and the topic behaves as a queue, with the work divided among them. Put each consumer in a group of its own and the same topic behaves as publish-subscribe, with every consumer seeing everything. Same broker, same topic, same events, and the documentation calls the arrangement a generalization of what traditional messaging systems do rather than a choice between their two models. So the honest version is that publish-subscribe and queue name two delivery semantics, selected per subscription, and not two kinds of product.

### Apache Kafka and the Kinesis family

**Apache Kafka** describes itself as an event streaming platform. Its unit is the topic, which its documentation compares to a folder whose events are the files, spread across partitions held by different brokers and replicated so that more than one broker carries a copy. The property that matters for the contract above is that consumption does not remove anything: in its own words, "you define for how long Kafka should retain your events through a per-topic configuration setting, after which old events will be discarded". That makes $R$ a per-topic setting and makes a topic a replayable log rather than a delivery buffer, which is what allows a new consumer to read history it was not present for.

**Amazon Kinesis Data Streams** is the managed equivalent on AWS, a stream of records held in shards that several applications read concurrently and independently of one another. Its retention window is the same parameter under another name: AWS documents a default of 24 hours, adjustable up to 8,760 hours, which is 365 days.

The Kinesis names have moved, and a pre-2023 reference will not match the console. Kinesis is a family rather than one product, and two members were renamed: Amazon Kinesis Data Analytics became Amazon Managed Service for Apache Flink in August 2023, and Amazon Kinesis Data Firehose became Amazon Data Firehose in February 2024, the latter keeping its APIs and endpoints unchanged. Kinesis Data Streams kept its name, and it is the member that plays the transport role described in this note, so a source written in or before 2022 that names "Amazon Kinesis" as a real-time transport means Kinesis Data Streams and not the two that were renamed.

Neither system is described here past what it is and which part of the contract it implements. Each has substantial documentation of its own, and this note is a reference for neither.

## Where it is used

[[Stream Processing]] is the computation that runs on what this mode carries, so the two are a transport and its workload rather than competing ideas: one moves the events, the other decides what is computed as they go past. [[Modes of Data Flow]] places this among the three ways two processes exchange data at all, and its table lines up the clauses above against the other two. [[Request-Driven Communication]] is the contract this one is defined by departing from, and the availability product stated there is the specific thing the broker is introduced to break.

[[Microservices]] is where both modes usually appear in one system, since a service that answers a query synchronously may also publish what it did for consumers it has never heard of, and the choice is made per interaction rather than per architecture. [[Distributed System]] supplies the failure model underneath all of it, including the reason a broker holding events is itself replicated across machines that can fail independently.
