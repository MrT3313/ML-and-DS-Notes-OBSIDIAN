---
note_kind: concept
aliases:
  - HTAP
  - hybrid transactional/analytical processing
  - hybrid transactional-analytical processing
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Some systems offer hybrid transactional/analytical processing, which aims to enable OLTP and analytics in a single system without requiring [[Extract-Transform-Load|ETL]] from one system into another. The pitch is that an aggregate can be computed over data that is current to the last commit, because there is no copy step and therefore no lag between the two halves.

Gartner created the term in early 2014, in a report on breaking the wall between transaction processing and analytics, and vendors adopted it from there rather than the other way round.

What is on offer is usually less of a merger than the name suggests. These systems often still have the same operational and analytical engines inside them, with a shared interface hidden in front. TiDB is the clearest published case: row-oriented storage in TiKV serves the transactional side, column-oriented storage in TiFlash serves the analytical side, the columnar replicas are kept in step asynchronously through Raft, and the query planner picks between the two per query on cost. The split did not go away. It moved below the SQL interface and became somebody else's operational problem instead of yours, which is a real benefit and a smaller one than "a single system" implies.

The same convergence is happening outside the category, in products nobody sells as HTAP, and the three usually named for it are worth separating because one of them is not the same kind of thing as the other two.

CockroachDB is the transactional side reaching toward analytics, and the supportable version of the claim is narrower than the pitch. It executes a query a column at a time rather than a row at a time, a vectorized engine sitting on top of row-oriented storage, which makes moderate analytical SQL tolerable without making the store columnar. Its own documentation still says it is not yet suitable for heavy analytics or OLAP, and points instead at change data capture as the route into an analytics engine, which is the copy step the HTAP pitch claims to remove.

DuckDB is the analytical side reaching toward transactions, and there the claim holds as stated. It is a columnar vectorized engine that documents full [[ACID]] transactions under its own multi-version concurrency control, guaranteeing snapshot isolation, which it maps to PostgreSQL's repeatable read. What bounds the claim is where it runs: in its embedded mode it is in-process rather than a server, and one process may write while many threads inside it work in parallel, or many processes may read with none writing. So "handles transactional queries" here means real transactional correctness for one application holding the file, not a shared operational store several services write to. A client-server protocol exists for it as of 2026 but is still beta and is not yet a settled fact about the product.

Apache Iceberg does not belong on the list at all, because it is not a database. It is an open table format, a specification for organizing immutable data files under a metadata tree, with no query engine and no catalog of its own: Spark, Trino, Flink and the rest supply the engine, and a separate catalog tracks which metadata file is currently the table. What makes it look like a transactional database is one mechanism its specification names. A commit is an atomic swap of the pointer to the current metadata file, and a writer that loses the race revalidates its assumptions and retries, which gives readers a consistent snapshot without taking a lock and gives writers serializable isolation over object storage. That is a genuine transactional property, and it belongs to a file layout rather than to an engine.

## Formal statement

**Not further quantitative at this depth.**

HTAP is a product category and an architectural goal rather than a measurable property, so there is no quantity attached to the term itself.

## Where it is used

It is the attempt to merge [[Online Transaction Processing]] and [[Online Analytical Processing]], the two workloads the rest of this folder keeps apart, and its architecture is best read as a comment on why they were kept apart in the first place.

It does not replace a [[Data Warehouse]], and the reasons are the ones that motivated the warehouse to begin with. A warehouse integrates data from many operational systems into one schema, so it answers questions no single operational database can; it keeps history that an operational store overwrites; and it isolates expensive scans from the traffic users are waiting on. Merging the two workloads inside one database addresses none of those three.
