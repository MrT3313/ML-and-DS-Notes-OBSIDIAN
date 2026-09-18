---
note_kind: concept
aliases:
  - data lake
  - data lakes
  - lake
  - sushi principle
up: "[[Online Analytical Processing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A data lake is a centralized repository holding a copy of any data that might be useful for analysis, obtained from the operational systems, and kept in the raw form those systems produced rather than transformed into some analysis schema first. A lake simply contains files, and it imposes no particular file format, data model or schema, so it can equally hold text, images, video, sensor readings, sparse matrices, feature vectors or genome sequences.

## VS

The difference from a [[Data Warehouse]] is when the schema is imposed: at write time there, at read time here. Deferring it means each consumer transforms the raw data into the form that best suits their needs, and two consumers can read the same bytes into two different shapes without either having to win an argument first. It also means nothing conforms the data on the way in, so every reader repeats the cleaning, and two readers who clean it differently produce two numbers that disagree with no error anywhere to say so.

The named form of the argument for deferring is the **sushi principle**, "raw data is better": data that has not been cooked can still be cooked any number of ways, and data that has been cooked cannot be uncooked. The phrase comes from a 2015 talk by Bobby Johnson and Joseph Adler, "The Sushi Principle: Raw Data Is Better", at Strata+Hadoop World, which is the citation [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]] attaches to it.

## Formal statement

A lake is a storage convention rather than a mathematical object, so the precise statement is about what it does and does not fix, and each half of it can be checked against a real repository.

It fixes exactly two things: that there is one place to look, and that the bytes found there are the ones the operational system produced. It fixes nothing else. In particular it fixes no file format, no data model and no schema. A lake can hold Avro, Parquet, JSON, CSV, log text and JPEGs side by side, and none of them is required by the concept. This is what the research literature calls schema-on-read, in contrast to the schema-on-write of a warehouse (Rihan Hai, Christos Koutras, Christoph Quix and Matthias Jarke, "Data Lakes: A Survey of Functions and Systems", *IEEE TKDE* 35(12), 2023).

The consequence worth stating plainly: because no schema is enforced on write, nothing in a lake makes two files describing the same entity agree with each other, and correctness is entirely the reader's problem.

## Where it is used

A lake is what you reach for when the consumers are not writing SQL. A warehouse's relational model queried through SQL fits business analysts well, and fits parts of data science work badly: reshaping columns into the form a model will be fitted on, which is [[Feature Engineering]], and the two jobs that start from unstructured input rather than rows, extracting structured information from text with natural language processing and from photographs with computer vision. Data scientists working on those tend to leave the relational database entirely and use Python libraries such as pandas and scikit-learn, statistical languages such as R, and distributed analytics frameworks such as Spark, all of which read files directly.

> [!note] How far the SQL claim goes
> "Bad for data scientists" is stronger than the evidence supports. Warehouses today can fit models from SQL without the data leaving them, ex. BigQuery ML, whose `CREATE MODEL` statement trains linear and logistic regression, k-means and matrix factorization in place. Dan Olteanu's "The Relational Data Borg Is Learning" (*PVLDB* 13(12), 2020), which [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]] lists among its references, argues further that a relational engine can exploit the structure of relational data to train faster than exporting the join result and fitting outside would. The defensible version is narrower: SQL over a conformed relational schema is an awkward substrate for that work, not an impossible one, and it stops being an option entirely once the input is a photograph.

Two structural relationships beyond that:

- [[Object Storage]] is what a lake is usually built on now. The lake is a pile of files, and services such as Amazon S3 are a pile of files that hides the machines underneath, which is the same shape at a price that makes keeping everything affordable.
- [[Extract-Transform-Load]] is how data arrives. A lake can be the destination, with each consumer transforming on read, or an intermediate stop on the path from the operational system to a [[Data Warehouse]], with the lake holding the raw landing copy and the warehouse holding the conformed one.

### File formats that turn up most

Neither is part of the concept, and a lake holding neither is still a lake. They are worth knowing because they are what the files usually are, and they differ in orientation.

- **Avro** is a row-oriented binary serialization format whose container file stores the writer's schema alongside the records, so a program that did not write the file can still read it.
- **Parquet** is a column-oriented file format, so a query touching three columns out of two hundred reads only those three off disk.
