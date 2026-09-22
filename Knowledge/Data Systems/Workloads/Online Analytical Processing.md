---
note_kind: concept
aliases:
  - OLAP
  - online analytical processing
  - analytical system
  - analytical systems
  - OLAP system
  - OLAP systems
  - OLAP workload
  - analytics workload
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Online analytical processing is the workload an analytical system carries: a query scans over a huge number of records and calculates aggregate statistics (count, sum, average, and the rest) rather than returning the individual records to the user. The query is a reduction: it consumes many records and emits few, usually one row per group, where an operational query emits roughly what it consumed. An *analytical system* is one that exists to serve the needs of business analysts and data scientists, who perform analytics and typically do not modify the data, though they might create derived datasets in which the original data has been processed in some way.

The questions such a query answers are questions about a population rather than about a row:

- What was the total revenue of each of our stores in January?
- How many more bananas than usual did we sell during our latest promotion?
- Which brand of baby food is most often purchased together with brand X diapers?

Unlike an operational workload, the queries are not known in advance. An analytical system gives users the freedom to write arbitrary queries by hand, or to generate queries automatically by using data visualization or dashboard tools: Tableau (Salesforce), Looker (Google Cloud), and Microsoft Power BI, which is now one of the core workloads of Microsoft Fabric, are the ones usually reached for. The consequence is that the system cannot pre-plan its access paths the way an OLTP database can, and has to be fast at scanning instead.

### Analytics embedded in a product

Not all of this workload is internal. *Product analytics*, also called real-time analytics, is the same analytical shape embedded into a user-facing product, so the person waiting on the aggregate is a customer rather than an analyst. Apache Pinot, Apache Druid, and ClickHouse are the systems built for it, and all three are current under those names: Pinot and Druid are top-level Apache projects, ClickHouse is an open-source column store with a hosted product alongside it.

What separates them from a warehouse-shaped analytical system is the ingest path and the latency target, not the workload class. They can ingest from a stream and make a record queryable as soon as it arrives, and they aim at sub-second responses, where a system fed by batch loading is tuned instead for getting a lot of work done per query and is content to answer in seconds or minutes. The distinction is a matter of degree rather than a boundary: each of these three describes itself as an OLAP system, and Pinot claims low latency and high throughput together rather than trading one for the other, so "real-time analytics" names a target these are optimized for and not a separate category of query.

## VS

The whole difference from [[Online Transaction Processing]] is what one request touches: an operational request reads or writes a handful of current records by key, an analytical one sweeps millions of historical records and returns a summary. The eight-row comparison of read pattern, write pattern, users, query set, query volume, what the data represents, and dataset size lives in [[Online Transaction Processing]] and is not repeated here.

The reduction is the property every downstream design choice follows from, columnar storage included, that being the layout set out in [[Row-Major and Column-Major Order]] and the one a query reading a few columns of very many rows wants. It is also why the dataset sizes reported in [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]] run to terabytes and petabytes: a system that answers questions about history has to keep the history.

"Online" in this name means interactive: the analyst asks a question and waits for the answer, rather than commissioning a report and receiving it next week. It is a different word from the "online" in [[Online Learning]], which names a model that updates its parameters one instance or mini-batch at a time.

The term fixes no latency target. "Interactive" is the only commitment it makes, and product analytics systems tighten that to sub-second by choice, not by definition.

## Formal statement

**Not further quantitative at this depth.**

Like its counterpart, OLAP describes the shape of a workload rather than a quantity, so there is no formula.

## Where it is used

The workload runs in one of two places: a [[Data Warehouse]], which holds a cleaned copy modelled for analysis, or a [[Data Lake]], which holds raw files and imposes structure only at query time. The data reaches either one through [[Extract-Transform-Load]] pipelines or a stream, which is also the reason an analytical system is almost always a consumer rather than a producer: nearly everything in it is [[Derived Data]], reconstructible from the operational systems it was copied out of, which is what makes it safe to rebuild.

[[Online Transaction Processing]] is the other half of the axis and the usual upstream source, the place the records were written before anyone asked a question about them in aggregate. [[Hybrid Transactional-Analytical Processing]] is the attempt to run both workloads in one system rather than copying between two.
