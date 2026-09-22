---
note_kind: concept
aliases:
  - structured data
  - unstructured data
  - semi-structured data
  - schemaless
  - schemaless data
up: "[[Data Model]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Structured data is data that has been committed to a predefined data model, which is to say a schema: the fields it has, what each one means, and what type each one holds, all decided before anything was written. Unstructured data is data that adheres to no such schema.

Both names live on this one note, because neither is definable except against the other. A reader who arrived here typing "unstructured data" is in the right place; the pair is one distinction, not two topics.

## VS

Committing to a schema means every stored value conforms to it, because nothing that fails to conform was accepted. The price is paid later and all at once: changing the schema is a change to the data already stored, so adding a field, splitting one, or tightening a type means going back over everything written under the old shape. There is no version of this where old data is left alone and new data uses the new schema, unless the readers are prepared to handle both, at which point the schema is doing less work than it claims.

Not committing means nothing was refused on the way in. A reader can still find intrinsic patterns and extract structure from them, and usually can: log lines from one program tend to have the same fields in the same order, and a folder of scanned forms tends to hold the same form. What is missing is the guarantee. There is no promise that every line follows the apparent pattern, and the failure mode is that the first line which does not follow it produces a wrong value rather than an error.

The middle case is **semi-structured** data: records that carry their own field names alongside their values, so each record describes its own shape without any shape being fixed in advance. JSON and XML are what this means in practice. A semi-structured record is self-describing, so a reader can find out what fields it has without external knowledge; what the reader still cannot find out is whether the next record has the same ones.

### Where structured data is kept, and where unstructured data ends up

[[Data Warehouse]] and [[Data Lake]] are already separated in this vault on when the schema is imposed, at write time in the one and at read time in the other. That is the sharper of the two available axes, and it is the one that stands. This note's structured-against-unstructured split is the upstream property rather than a competing axis, and the two line up like this.

When the schema is imposed is the more precise statement; whether the data has one at all is the upstream property. A warehouse imposes a schema on write, so by construction it holds structured data. A lake imposes none, so it can hold either, which is why a lake is where unstructured data goes and not what makes it a lake.

The asymmetry is the part worth keeping. "A warehouse holds structured data" is an entailment, since imposing a schema on write is what producing structured data means. "A lake holds unstructured data" is not an entailment but an observation about what people put there, and a lake full of Parquet files written against a stable schema is still a lake.

## Formal statement

Having a schema is a contract, and it can be stated exactly enough to be found false.

For structured data, the schema fixes three things: the set of fields, the type of each field, and the guarantee that **every** record in the collection satisfies both. The third is the one doing the work. Because it holds for every record, a query written against the schema is well defined over the whole collection without anything inspecting the records first: asking for the mean of a numeric field is meaningful because the field exists everywhere and is numeric everywhere. A violation is a violation of the contract, and it was refused at write time by whoever enforces the schema, with the writer present to see the refusal.

For unstructured data none of the three is fixed. A reader may still assume a set of fields and a type for each, and that assumption is a hypothesis about the data rather than a property of it. Nothing tested it when the data was written and nothing tests it when the data is read, so the failure is silent: a record that does not match produces a parse that succeeds on the wrong interpretation, a field read from the wrong position, or a value quietly skipped, and the number that comes out at the end is wrong with no error anywhere in the path to say so.

Semi-structured data fixes the first of the three per record and none of them across records. Each record names its own fields, so a reader need not guess what is in the record in front of it, and still has no guarantee about the next one. That is why a schemaless store shifts the responsibility for assuming structure from the writing application to the reading application rather than removing it.

The checkable form of all this: given a collection and a proposed schema, structured data admits a decision procedure, since every record can be tested against the schema and one failure refutes the claim. For unstructured data the same test is available and means less, because passing on the records present says nothing about the records to come.

## Where it is used

[[Data Warehouse]] is the repository that holds processed data conformed to a schema and ready to be used, which is the structured half of this note made concrete.

[[Data Lake]] is the repository that holds raw data before processing, which is where unstructured data goes for the reason given above, and the reason its readers repeat the cleaning that a warehouse does once.

[[Data Model]] is what "predefined data model" points at. Structured data is data committed to one, so the choice of model is what a schema is written against, and the structured half of this distinction is what makes a model enforceable at all.

[[Relational Model]] is the strict case. A relational database requires every row to conform before it is accepted, which makes it the most demanding consumer of structured data in the vault and the place where the retrospective-update cost is felt hardest.

[[Data Serialization]] is what the distinction is often confused with. A format can carry field names with the values, as JSON and XML do, which makes records self-describing; that is a property of the encoding and it does not supply a schema, because self-describing records may still describe entirely different things.
