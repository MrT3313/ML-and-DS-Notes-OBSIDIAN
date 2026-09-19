---
note_kind: index
aliases:
  - data system
  - data systems
  - data infrastructure
up: "[[Home]]"
---

The machinery that stores, moves and serves data, kept as its own domain because every note in it would still be true if no model were ever fitted on anything it holds. That is the membership test, and it is the whole boundary: a note earns a place here when it is about where the data lives, how it gets there, who answers a query about it, and what breaks on the way. `Knowledge/Machine Learning/` owns what is learned from data; this domain owns the substrate that learning reads out of, which was built for reasons that have nothing to do with learning and would be built anyway. `Knowledge/Machine Learning/Operations/` owns operating a model that has shipped; this domain owns operating the services and the infrastructure underneath it, work that exists whether or not a model is among the things being operated. A note that only makes sense because there is a fitted model somewhere in the picture belongs to that domain and not here. `Knowledge/Systems Architecture/`, indexed at [[Systems Architecture]], is the domain this one leans on, and it owns the concepts that stand for any software system regardless of what it stores: how work is spread across machines, who owns the machines, and who operates them. The domain is one chapter old. All thirteen notes come from [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]], which surveys the architectural choices rather than the mechanisms, so what is on disk is the vocabulary and the shape of the trade-offs rather than how any single system works inside.

## Areas

- **`_Foundations/`** the vocabulary the rest of the domain is stated in. [[Data-Intensive Application]] is the axis the whole thing is organized around, an application whose primary difficulty is data management rather than arithmetic, with its four recurring challenges and the compute-intensive counterpart it is defined against. [[System of Record]] and [[Derived Data]] are the authority pair: the first holds the canonical copy and wins any disagreement by definition, the second is anything computable from it, redundant on purpose and rebuildable by rerunning the transformation, with the cache as the worked example of why a copy is allowed to exist only when a fallback path does.
- **`Workloads/`** the shape of the traffic a system is built to take, which is the split every later choice follows from. [[Online Transaction Processing]] is many small requests touching few records each, answered while somebody waits, and it carries the eight-row comparison table that the whole folder turns on. [[Online Analytical Processing]] is the other half, few queries each sweeping millions of historical records and returning a summary, including the product-analytics variant that tightens the latency target without changing the workload class. [[Point Query]] is the exact-match lookup by key that defines the first of those, and [[Transaction]] is the bracket an application draws around a group of reads and writes so the database's guarantees attach to the group rather than the statements. [[Hybrid Transactional-Analytical Processing]] is the attempt to put both workloads back in one system, and its internals mostly explain why they were separated.
- **`Analytics/`** where the copy goes once it has left the operational systems. [[Data Warehouse]] is the separate read-only store, conformed to one analysis schema on write and non-volatile so history accumulates instead of being overwritten, with a precise account of what it guarantees (isolation, one schema over many sources, rebuildability) and what it does not (freshness, authority, completeness). [[Data Lake]] is the same copy one step earlier, raw files with structure imposed at read time, defended by the sushi principle and undercut by the fact that nothing makes two files agree. [[Data Silo]] is the condition the warehouse exists to answer, one entity living in several systems under several identifiers with nothing holding the correspondence. [[Extract-Transform-Load]] is the running machinery of the copy, the only `method` note in the domain, carrying both letter orderings, what moves when the transform moves, and a failure list led by the upstream schema change nobody told the pipeline about.
- **`Operations/`** the role that joins the two halves of the domain to each other. [[Data Engineering]] is the work of integrating the operational and analytical systems and owning the pipelines, schedules and storage that integration runs on, described as a pair of roles rather than a technique: the data engineer, who owns the movement of data out of the system of record and the infrastructure it moves over, and the analytics engineer, a title datable to 2019, who applies software engineering discipline to the transformation code itself so it is version controlled, tested, documented and deployed rather than passed between people as SQL.

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```
