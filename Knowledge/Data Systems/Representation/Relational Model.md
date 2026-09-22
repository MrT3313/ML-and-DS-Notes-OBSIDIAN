---
note_kind: concept
aliases:
  - relational model
  - relational
  - relation
  - relations
  - relational database
  - relational databases
  - SQL
  - structured query language
up: "[[Data Model]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

The relational model organizes data into relations, where a relation is a set of tuples. A table is the accepted way of drawing a relation, one tuple per row and one attribute per column, but the table is the picture and the relation is the thing. A database built on this model is a relational database, and the data in one is fetched through a query language, in practice SQL.

## Formal statement

E. F. Codd's "A Relational Model of Data for Large Shared Data Banks", *Communications of the ACM* 13(6), 1970, is the primary source, and it does define a relation as a subset of a Cartesian product. Given domains $D_1, D_2, \dots, D_n$, not necessarily distinct, a relation $r$ of degree $n$ is

$$r \subseteq D_1 \times D_2 \times \cdots \times D_n$$

so $r$ is a set of $n$-tuples, each of whose $i$-th component is drawn from $D_i$. The number of tuples is the cardinality of the relation and the number of domains is its degree.

### What follows from "set"

A set has no order and no duplicates, and both facts are load-bearing.

Row order carries no information. If $t_1$ and $t_2$ are tuples then $\{t_1, t_2\} = \{t_2, t_1\}$, so a relation listed in one order and the same relation listed in another are the same relation, not two. Any ordering seen in a result came from the query asking for one, never from the relation having one.

Attribute order carries no information either, though this takes one more step to get to honestly. In Codd's Cartesian-product construction the position of a domain is part of the tuple, and he says so; he then observes in the same paper that users should not be made to depend on that ordering and introduces naming so that attributes can be addressed by name instead. The standard modern formulation finishes the job by making a tuple a mapping from attribute names to values,

$$t : \{A_1, \dots, A_n\} \to D_1 \cup \cdots \cup D_n, \qquad t(A_i) \in D_i$$

under which there is no position to shuffle. Either way the observation holds: shuffle the rows, shuffle the columns, and the relation is unchanged.

What that buys and what it costs is one trade. Because position carries nothing, the query language may not depend on position, and so it addresses attributes by name. The cost is that "the third column" is not a thing you can ask for. The gain is that a query naming the attributes it wants keeps working when a column is added beside them, and two relations can be compared on the attributes they share without either having to be laid out to match the other.

### Declarativity and what it is worth

SQL is a declarative language: the statement says which output is wanted, and the system works out the steps that produce it. What fills the gap is the query planner, which turns the statement into a query plan, a concrete sequence of scans, index lookups, joins and aggregations, and the query optimizer, which chooses among the plans that would all produce the same answer.

The property that makes declarativity pay is exactly that the statement does not fix the steps. Because the steps are not in the query, the optimizer is free to change them without the query changing, so the same unedited query can get faster when an index is added, when the data grows, or when the engine learns a better join strategy. A procedural formulation that spelled out the steps would have to be rewritten each time for the same gain.

### What the model demands in return

A relational database requires a strict schema, and every stored row must conform to it. The consequence is that changing the schema is a change to data already written, not only to future writes, which is why schema management is painful in a way that has no counterpart in a schemaless store. [[Structured Data]] carries what committing to a schema fixes and what it costs.

The second cost is the join. [[Database Normalization]] deliberately spreads one entity's facts across several relations so that each fact is stored once, and putting them back together is a join. A join matches tuples across relations, so its cost scales with the sizes of the relations being matched rather than with the size of the answer, and on large relations that is expensive enough to be the thing a query is built around.

Relations are typically stored in file formats such as CSV or Parquet, which is a separate choice from the model: the model says the data is a set of tuples, and [[Data Serialization]] says what the bytes look like, row by row in the CSV case and column by column in the Parquet case.

The query language half of this note is a sketch. [[DDIA]] chapter 3, "Data Models and Query Languages", is what will deepen both halves, and it is titled for exactly this subject.

## Where it is used

[[Database Normalization]] is the design theory that only exists because of this model: it is stated in terms of relations and functional dependencies among their attributes, and it is what produces the joins above.

[[Data Model]] is the parent question, where the relational model sits against the document and graph families on what a unit of storage is and how one unit refers to another.

[[Online Transaction Processing]] is the workload relational databases were built for, small requests reading and writing individual records by key, and it is where the strict schema pays for itself because the application can rely on every row being shaped the way it expects.

[[Data Warehouse]] is the relational model used for the other workload. A warehouse's schema is usually dimensional, a large fact table surrounded by descriptive tables, and that is a relational schema chosen for analysis rather than a departure from the model.

[[Structured Data]] is the property the strict schema requires. Data with a predefined schema is what a relation can hold, and data without one has to be given a shape before it can be stored relationally at all.
