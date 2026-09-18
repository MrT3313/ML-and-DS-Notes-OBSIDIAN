---
note_kind: method
aliases:
  - ETL
  - ELT
  - extract-transform-load
  - extract transform load
  - extract-load-transform
  - extract load transform
  - data pipeline
  - data pipelines
  - ETL pipeline
  - ELT pipeline
up: "[[Data Warehouse]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## What it does and when

An ETL pipeline is the scheduled machinery that copies data out of the systems that produced it and puts it, reshaped, somewhere analysts can query. Data is extracted from the operational databases, transformed into an analysis-friendly schema, cleaned up, and loaded into the destination. The usual source is a set of [[Online Transaction Processing]] databases and the usual destination is a [[Data Warehouse]]; a [[Data Lake]] can be the destination instead, or an intermediate stop on the path from the operational system to the warehouse, holding the raw landing copy while the warehouse holds the conformed one.

Reach for one whenever analysis has to read data that lives somewhere it must not be queried from, which is the normal case: the four reasons an analyst should not hit an operational database directly are exactly the reasons this pipeline exists. It is bulk, scheduled work, not request-response work, and its unit of output is a batch of rows rather than an answer to a question.

> [!warning]
> This is the data-engineering sense of the word "pipeline", not the one [[Pipeline]] carries. A `sklearn.pipeline.Pipeline` chains estimator calls inside a single process, handing an array straight from one object's `fit_transform` to the next object's `fit`, and the whole chain lives and dies within one `fit` call. Here the stages are separate programs, usually separate machines, run on a schedule, and the interface between them is storage: a stage writes its output to a table or a file and a later stage reads it back, possibly hours later, which is what lets a stage fail and be rerun without the stages around it being rerun too. The two share the word and nothing else.

## Algorithm

The letters name the order of three steps. Extract-transform-load:

1. **Extract.** Read the rows out of each source system, either as a periodic snapshot or as a feed of the changes since the last run. This is the step that has to be gentle with the source, since the source is a live operational database.
2. **Transform.** Reshape into the destination's analysis schema: conform identifiers so that a customer in the billing system and a customer in the support system become the same key, cast types, fix units, deduplicate, clean, aggregate. This runs before anything is written to the destination, on compute that the destination does not own.
3. **Load.** Write the transformed result into the destination, normally in bulk.

Extract-load-transform is the same three steps in a different order:

1. **Extract.** Identical. Nothing about this step changes.
2. **Load.** Write the records into the destination as they came out of the source, with no reshaping.
3. **Transform.** Run the same reshaping as queries executed by the destination's own engine, producing derived tables that sit alongside the raw ones.

The only thing that moves between the two is where the transform runs and when. That single move has consequences on both sides. Under ETL the destination only ever holds data that passed the transform, so it stays small and everything in it has a known meaning, but a change to the transform means going back to the source and extracting again, and whatever the transform discarded is gone. Under ELT the raw record is already in the destination, so re-running the transform costs no second extract and a question the original schema did not anticipate can often be answered from data already sitting there; the costs are storing the raw copy and paying the destination for the transform compute.

One correction to a common way of stating the distinction: ETL is often described as transforming "on a separate processing server", and that is the typical deployment rather than the definition. What separates the two orderings is whether the data is transformed before or after it lands in the target, not which machine does the work. Even the classical warehouse architecture never put all of the transformation outside: Chaudhuri and Dayal's 1997 survey describes the load step itself as still doing integrity checking, sorting, summarization and aggregation to build the warehouse's derived tables. And in the other direction, the tool that defines the modern transform step runs its SQL inside the destination and owns no server of its own.

## Hyperparameters

A pipeline has a few, and they are switches rather than dials, so "increasing" below means moving from the weaker guarantee to the stronger one. Each is a setting one of the systems under `## Implementation` exposes.

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| Load mode, incremental or full rebuild | dbt `materialized` (`view`, `table`, `incremental`); Airbyte incremental versus full refresh sync | dbt: `view`; Airbyte: chosen per stream | A full rebuild re-reads the whole source every run, so a row deleted at the source disappears from the target; an incremental load keeps the rows earlier runs delivered, so the deletion is never reflected unless a change feed or a soft-delete flag (Fivetran's `_fivetran_deleted`) carries it across | Full rebuild while the source is small enough to re-read on every run; incremental once it is not, with the deletion path decided at the same time |
| Deduplication key on an incremental load | dbt `unique_key` with `incremental_strategy` (`append`, `merge`, `delete+insert`, `insert_overwrite`); Airbyte "Incremental, Append" versus "Incremental, Append + Deduped" | dbt: no key, with `append` on Postgres and `merge` on Snowflake and BigQuery; Airbyte: chosen per stream | Without a key a rerun appends a second copy of every row it re-delivers, so the target holds each row at least once; with a key and a merging strategy exactly one row per key survives and a rerun is idempotent | Set a key on any incremental model whose source can re-deliver rows, which is any source that can be backfilled; leave `append` for immutable event streams only |
| Change capture, cursor or log | Airbyte incremental on a cursor field versus log-based CDC; Fivetran log-based replication for database sources | Cursor-based unless the connector supports CDC and it is switched on | A cursor sees a row only when its cursor column moves, so hard deletes and every intermediate update between two runs are missed; the log carries each update in order and the deletes with them | Log-based whenever the source exposes its log and the target must reflect deletions; cursor-based where it does not, or where deletions do not matter |
| Backfill of missed intervals | Airflow `catchup` | `True` in Airflow 2, `False` in Airflow 3 | `True` runs one DAG run for every interval between the start date and now, so rows for intervals the scheduler was down for still arrive; `False` runs only the latest interval and those rows never arrive unless somebody backfills by hand | `True` when the target must hold every interval's rows; `False` when each run is a full rebuild and the missed intervals add nothing |

Schedule, batch size, parallelism, worker count, retry count and timeouts do not pass: they change when and how fast the rows arrive, and whether a run finishes tonight or tomorrow, never which rows arrive or what is in them.

## Failure modes

- **A schema change upstream.** Someone on the application team adds, renames or repurposes a column in the operational database. They have no reason to know the pipeline exists. The extract keeps running and the transform either fails on a type it did not expect or, worse, silently drops the column and produces a table that looks fine.
- **The extract competing with the application.** A full-table scan against the primary is precisely the load the warehouse was built to keep off the operational system. A long extract holding reads on a busy table degrades the application it is copying from, which turns the pipeline into the problem it was supposed to solve.
- **A partial load.** A run that writes the fact rows and dies before the rows they refer to leaves a destination that still answers queries, wrongly. Chaudhuri and Dayal's 1997 survey already treats this as the load utility's central requirement: it must be possible to monitor, suspend, resume and restart a load without loss of integrity, which means checkpointing rather than hoping.
- **A backfill that is not idempotent.** Rerunning yesterday's job after fixing a bug appends a second copy of every row unless the load is keyed to replace rather than insert. The damage is silent and shows up as doubled totals weeks later.
- **The transform becoming the only place a definition is written down.** Once "active customer" exists only as a `WHERE` clause in a transform, two pipelines that implement it differently reproduce inside the warehouse exactly the inconsistent metrics that a [[Data Silo]] caused outside it.
- **Staleness read as fact.** Consumers see a copy, so a dashboard is as old as the last successful run. A job that failed overnight is indistinguishable from a quiet night in the data unless something separate checks that the run happened.

## Implementation

There is no single library here, and no code worth pinning: this is a shape that several classes of system implement between them. What separates them is two things, whether the tool orchestrates or transforms, and where the transform's compute lives.

**Orchestration.** [Apache Airflow](https://airflow.apache.org/) calls itself a platform for developing, scheduling and monitoring workflows, and specifically a platform for orchestrating batch workflows. It decides what runs, in what order, after what, and what happens on failure. It does not move or transform the data itself; the tasks it triggers do, and Airflow owns no data-processing compute. Dagster ("a data orchestrator built for data engineers", organized around assets and their lineage rather than around tasks) and Prefect ("an open-source orchestration engine that turns your Python functions into production-grade data pipelines") fill the same slot with different programming models.

**Managed extract and load.** Fivetran and Airbyte supply the connectors so nobody has to write and maintain the extract for each source by hand: Fivetran as a proprietary managed service advertising more than 700 connectors plus a managed data lake destination in open table formats, Airbyte as an open-source replication platform with more than 600 sources and destinations. Both do the E and the L and stop there. Fivetran's own documentation is explicit that it does not run the transform itself, but either runs it in the destination through dbt or triggers dbt Cloud to run it.

**In-warehouse transform.** [dbt](https://docs.getdbt.com/) is the T of ELT and nothing else. You write `SELECT` statements, dbt compiles them into a dependency graph of models and hands the SQL to the destination to execute, so the transform's compute is the warehouse's compute and dbt itself is a code generator and scheduler for SQL. It does not extract and it does not load.

**Distributed transform.** [Apache Spark](https://spark.apache.org/) describes itself as a unified analytics engine for large-scale data processing, with SQL and DataFrame APIs over a cluster it runs on itself (standalone, YARN, or Kubernetes). It is the transform that does not live in a warehouse: it reads files out of a distributed filesystem or object store, which is what you need when the input is a [[Data Lake]] of Parquet and JSON rather than a relational schema.

Placed on those two axes: Airflow, Dagster and Prefect orchestrate and own no data compute. Fivetran and Airbyte own the extract and load and no transform. dbt owns the transform and borrows the destination's compute. Spark owns the transform and brings its own.
