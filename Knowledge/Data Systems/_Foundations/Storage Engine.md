---
note_kind: concept
aliases:
  - storage engine
  - storage engines
  - database engine
  - database engines
up: "[[Data Systems]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

A storage engine is the component that implements how data is actually stored on a machine and how it is retrieved again. It owns the bytes on disk and the paths that reach them, and nothing else.

Calling a storage engine a database is close enough to be worth sharpening rather than repeating. A database is the whole product somebody installs, and the engine is one layer inside it. Deciding what the data is allowed to look like and accepting a query about it happen above that layer, which is why the same server can be given a different engine and still speak the same language to the same application.

## Formal statement

The engine is separable from the query interface above it. That is the content of the term, and two kinds of system make the separation checkable rather than merely plausible.

**One interface, several engines.** MySQL is built on a pluggable storage engine architecture: engines can be loaded and unloaded from a running server, the application reaches the server through connector APIs and a common service layer that sits above them, and the engines are the components that perform the actions on the physical data underneath. The choice is made per table, not per installation. MySQL's own documentation states that you are not restricted to using the same storage engine for an entire server or schema and can specify the engine for any table. What changes when it is switched is the guarantees, not the dialect: InnoDB, the default, is transaction-safe with commit, rollback and crash recovery, locks at row granularity, and enforces foreign keys, while MyISAM supports no transactions and locks a whole table at a time, which is why it is used for read-only and read-mostly tables. The same `UPDATE` statement is accepted by both and means something weaker under one of them.

**An engine with no interface at all.** LevelDB and RocksDB ship as libraries embedded into an application rather than as servers. LevelDB's own documentation says it is not an SQL database, has no relational data model, supports no SQL queries and has no support for indexes; keys and values are arbitrary byte arrays, data is stored sorted by key, and the operations are `Put(key, value)`, `Get(key)`, `Delete(key)`, atomic batches of those, and forward and backward iteration. RocksDB describes itself as an embeddable persistent key-value store for fast storage, is built on the earlier LevelDB work, and treats keys and values the same way. A storage engine is therefore a thing you can have without any query language present, which is the strongest available evidence that the two are separate layers.

**The contract that implies.** The engine owns the on-disk representation and the access paths. What it owes the layer above is small: put a record, get one back by its key, and scan in key order. Everything the layer above does, parsing, planning, joining, aggregating, is built out of those three.

**What it does not fix.** The data model and the query language are decided above it, which is the reason one engine can sit under more than one. A key-value library underneath a relational server and the same library underneath a document store are the same engine serving two different models, and a table swapped from one MySQL engine to another keeps its columns and its SQL.

**What it is tuned for.** A storage engine is optimized for one of two workload shapes, [[Online Transaction Processing]] or [[Online Analytical Processing]], and an engine built for one is bad at the other. The mechanism that ties the engine to the workload is the physical layout it writes, which is [[Row-Major and Column-Major Order]]: keeping a record's fields together serves a request that wants whole records by key, keeping a column's values together serves a query that scans one field across millions of records.

Indexes, the structures underneath them, and how a lookup actually finds a row are not reached here. They are the subject of [[DDIA]] chapter 4, "Storage and Retrieval", which is titled for exactly this component, and this note stops at its boundary.

## Where it is used

[[Online Transaction Processing]] and [[Online Analytical Processing]] are the two workloads an engine is built for, and the split between them is the main reason more than one engine exists rather than one good one. [[Row-Major and Column-Major Order]] is the layout decision that connects the two: it is the concrete thing an engine does differently depending on which workload it was built to serve.

[[Data Model]] is the question the engine does not answer, which is why the two are asked separately: the model says what a record is and how it may be queried, the engine says how those records are put on disk, and either can be changed while the other stays.

[[Data-Intensive Application]] is where the choice matters, since storing and processing large data volumes is one of the four things that makes an application data-intensive, and the engine is the component that carries that job.
