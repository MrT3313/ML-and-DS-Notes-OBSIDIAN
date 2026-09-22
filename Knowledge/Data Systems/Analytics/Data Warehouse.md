---
note_kind: concept
aliases:
  - data warehouse
  - data warehouses
  - warehouse
  - data warehousing
  - enterprise data warehouse
  - EDW
up: "[[Online Analytical Processing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

A data warehouse is a separate database holding a read-only copy of the data from an organization's operational systems, so that analysts can query it without affecting those systems. The copy is not a mirror: on the way in the data is conformed to one schema chosen for analysis, cleaned, and only then stored, which means the warehouse commits to what the columns mean before anything is written.

The schema that gets chosen is usually dimensional, a large table of events (sales, clicks, orders) surrounded by smaller tables describing the things those events refer to, the arrangement known as a star schema. Chaudhuri and Dayal's 1997 survey quotes the older and still standard definition from Inmon:

> a "subject-oriented, integrated, time-varying, non-volatile collection of data that is used primarily in organizational decision making"
>
> W. H. Inmon, *Building the Data Warehouse*, 1992, as quoted in Surajit Chaudhuri and Umeshwar Dayal, "An Overview of Data Warehousing and OLAP Technology", *ACM SIGMOD Record* 26(1), 1997.

"Non-volatile" is the part that is easy to miss and does most of the work: rows arrive and are not updated in place afterwards, which is why the warehouse accumulates a history rather than a current state.

## VS

A warehouse imposes its schema at write time; a [[Data Lake]] imposes one at read time. That single difference sets everything else about the two, including the coarser way the pair is often split: imposing a schema on write is what producing [[Structured Data]] means, so a warehouse holds structured data by construction rather than by a second axis. Because the warehouse decides once, and decides for every reader, every consumer gets the same conformed columns with the same meanings, and the storage engine can lay out, partition and index for the queries those columns permit, which in practice means keeping them in [[Row-Major and Column-Major Order|column-major order]]. The price is that a question the schema did not anticipate is not merely slow, it is unanswerable: whatever the transform dropped is not in the warehouse at all, and getting it back means changing the pipeline and reloading from the source. The lake note carries the other side of the trade.

## Formal statement

A warehouse is a contract about a copy, and the two halves of it are read very differently: the first is why anyone builds one, the second is what people forget and then argue about.

It guarantees:

- Reads against it do not contend with the operational workload. That isolation is the whole reason it exists, and it is a property of the copy being separate, not of any query being fast.
- One schema over several sources. Data that arrived from different operational systems has been made to agree on identifiers, types and units before storage, so a single query can span them.
- Rebuildability. The content is [[Derived Data]]: every row came from somewhere else, so a warehouse that is lost or found to be wrong can be reconstructed by re-running the pipeline against the sources.

It does not guarantee:

- Freshness. The copy is exactly as current as the last successful load, and nothing about querying it reveals when that was.
- Authority. If the warehouse and an operational system disagree, the operational system is right by definition, because it is the [[System of Record]] and the warehouse is a copy of it.
- Completeness. Only what the schema and the transform chose to carry is present, so the absence of a fact in the warehouse is not evidence that the fact does not exist.

## Where it is used

It exists to keep analysts off the [[Online Transaction Processing]] systems. Querying those directly is generally undesirable for four separate reasons, and a warehouse answers all four at once:

- The data of interest can be spread across multiple operational systems, making it hard to combine datasets into a single query. That is a [[Data Silo]], and consolidating the sources into one queryable copy is the warehouse's answer to it.
- The kinds of schemas and data layouts that are good for OLTP are less well suited for analytics.
- Analytical queries can be very expensive, and running them over an operational database can impact the performance of other users.
- Operational systems may sit in a separate network that analysts are not allowed to reach directly, for security or compliance reasons.

The workload it is built for is [[Online Analytical Processing]]: few queries, each scanning many rows and returning aggregates rather than records. Data reaches it through an [[Extract-Transform-Load]] pipeline, which extracts from the operational databases, transforms into the analysis-friendly schema, cleans up, and loads the result. A [[Data Lake]] is the alternative destination when the consumers are not writing SQL, and can also sit in front of the warehouse as an intermediate stop.
