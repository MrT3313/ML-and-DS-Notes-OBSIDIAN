---
note_kind: concept
aliases:
  - data model
  - data models
  - NoSQL
  - non-relational
  - document model
  - document database
  - document databases
  - graph model
  - graph database
  - graph databases
  - graph data model
up: "[[Data Systems]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

A data model is the shape data is organized into: what a stored unit is, how one unit refers to another, and what a query is allowed to ask for. It is chosen by a designer before anything is written, and the choice decides how the system around it gets built and which problems that system can answer at all. That is the whole thesis of the note. A question the model cannot express is not a slow question, it is one the application has to answer by reading the data out and doing the work itself.

This is a different thing from the vault's [[Model]], which is a function from a feature vector to an output together with the parameters fitted to it. A data model is picked in advance and constrains what may be stored; a [[Model]] is estimated from what was stored and constrains nothing. Neither one illuminates the other, and a reader who arrived here wanting the fitted function wants that note.

Three families are in general use: the relational model, which has its own note at [[Relational Model]], and the two families usually grouped as NoSQL, the document model and the graph model.

### What NoSQL names

The word is younger than the systems it covers and names none of their properties. Carlo Strozzi used "NoSQL" in 1998 for a relational database that simply had no SQL interface, which is not the modern sense. The modern sense starts with a meetup on 11 June 2009 in San Francisco, organized by Johan Oskarsson to discuss open source distributed non-relational databases; the organizers needed something short for a Twitter hashtag, and "NoSQL" was what they settled on. Martin Fowler's account lists the systems that presented there, Voldemort, Cassandra, Dynomite, HBase, Hypertable, CouchDB and MongoDB, and the term spread from that room outward. The later gloss "Not Only SQL" is a retrofit onto a hashtag, not the meaning it was coined with.

The consequence matters more than the trivia. The word says what a store does not use, so membership in the category implies nothing about what a store does. The document model and the graph model are both NoSQL and have almost nothing in common with each other: one is built for entities that rarely refer to each other, the other for data that is mostly references. Reasoning from "it is NoSQL" to any shared property is reasoning from a negation, and it does not go through.

## Formal statement

Each family is a set of commitments, and the commitments are checkable against any real store claiming membership. The five axes below are the ones that decide what an application built on the model has to do for itself.

| Axis | Relational | Document | Graph |
| --- | --- | --- | --- |
| Unit of storage | a tuple in a relation, all its attributes flat | a self-contained document in a collection, nested values allowed | a vertex carrying its own properties |
| How one unit refers to another | an attribute value that matches a key of another relation, resolved when the query runs | nesting the referent inside the same document, or holding an identifier the reader must chase itself | an edge, which is stored as a record of its own and names both ends |
| Where the schema is enforced | on write, by the database, for every row before it is accepted | on read, by the application doing the reading; two documents in one collection may share no fields | on read for the properties; the store enforces only that an edge has two ends that exist |
| What a cross-unit join costs | a join the query planner performs, with cost scaling in the sizes of the relations being matched rather than in the size of the answer | no join operator to call: the application issues further reads and stitches the result, or the data was duplicated into the document to avoid it | following an edge is a hop from one vertex to the next, so cost scales with the number of edges walked and not with how much else is stored |
| Access pattern it is fast at | conditions stated over whole relations, including questions nobody anticipated | fetching one entire entity by its identifier | finding what is reachable from a starting vertex across many hops |

Two rows carry most of the consequences. Schema enforcement on write means a violation is refused at the point of writing, with the writer present to see the error; enforcement on read means a violation is discovered later, by whoever reads it, possibly as a wrong answer rather than an error. And the join row is the reason locality is a real property and not a preference: a model that keeps a whole entity in one place is fast at reading that entity and has no mechanism for relating it to another, while a model that splits entities across relations pays on every read and can relate anything to anything.

### Document model

Data arrives as self-contained documents, and references from one document to another are rare by assumption rather than forbidden by the model. A document is usually one continuous string encoded as JSON or XML, or in a binary form such as BSON, and every document in a store is assumed to use the same encoding. A collection is roughly what a table is and a document is roughly what a row is, with one difference that changes everything downstream: every row in a table obeys the table's schema, while two documents in one collection may have entirely different fields. That is what schemaless means, and it is a transfer of work rather than a removal of it. The writing application no longer has to make its records fit a declared structure; the reading application now has to assume a structure that nothing guarantees.

The gain is locality. One document holds everything about its entity, so reading the entity is one read, where a relational store may have the same entity spread over several relations. The loss is on the other side of the same fact: joining across documents is harder and less efficient than joining across relations, because there is no join to ask for.

That is as far as the current reading goes. [[DDIA]] chapter 3, "Data Models and Query Languages", is what will deepen it, and the question left standing is what happens when the relationships turn out not to be rare after all.

### Graph model

Reach for a graph model when relationships between items are common and important rather than incidental. The data is vertices and edges, and an edge is the relationship, stored in its own right. The contrast with the document model is a contrast of priorities: a document store treats the content of each document as the thing worth keeping close, and a graph store treats the connections as the thing worth keeping close, which is why retrieval framed as a question about relationships is fast here and slow elsewhere.

A vertex in a graph is not a [[Node]] in the sense this vault uses that word elsewhere. A [[Node]] is one participant in a distributed system, and it must hold private state, keep its own clock, be reachable only by messages and fail independently of the others. A graph vertex is a stored record with none of those properties and is not a machine. The overlap is the word only, and a graph with millions of vertices may be held on a single [[Node]].

Two graph models are actually in use, and they differ in what a stored fact looks like:

- The **property graph**, where vertices and edges both carry key-value properties and edges are typed and directed. Neo4j is the standard implementation, queried in Cypher.
- The **triple store**, where every fact is a subject, a predicate and an object, which is the RDF model from the semantic web. Apache Jena and Amazon Neptune implement it, queried in SPARQL. Neptune is a useful case because it serves both models in one service, property graphs through Gremlin and openCypher and RDF through SPARQL, which shows the two are alternative encodings of graph-shaped data rather than different subjects.

Full treatment waits on [[DDIA]] chapter 3, "Data Models and Query Languages", which covers both models and their query languages.

## Where it is used

[[Relational Model]] is the family this note only sketches, and it holds the precise account of relations, the set semantics underneath them and the query language built on top. It is also the only one of the three with a normalization theory, at [[Database Normalization]].

[[Structured Data]] is the upstream property the schema column of the table depends on: enforcing a schema on write is only possible for data that has one, and the structured against unstructured split is what decides whether a model can be imposed at all.

[[Storage Engine]] is the other half of every row in the table. A data model says what may be stored and asked for; a storage engine decides how the bytes are laid out and retrieved, and the same model can sit on very different engines with very different performance.

[[Data Serialization]] is where the logical shape meets the bytes. A data model is the shape, a serialization format is the encoding, and the two cross constantly: a document is a logical unit of the document model and is also, on disk, a JSON string. Picking a model does not pick a format and picking a format does not pick a model.

[[Model]] is the disambiguation link, held here so that a reader who followed the word "model" from machine learning material lands somewhere with a way out.
