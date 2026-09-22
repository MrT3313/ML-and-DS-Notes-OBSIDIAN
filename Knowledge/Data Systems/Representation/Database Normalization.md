---
note_kind: concept
aliases:
  - database normalization
  - database normalisation
  - normal form
  - normal forms
  - first normal form
  - 1NF
  - second normal form
  - 2NF
  - third normal form
  - 3NF
  - Boyce-Codd normal form
  - BCNF
  - functional dependency
  - denormalization
up: "[[Relational Model]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Database normalization is the design discipline of decomposing relations until every fact is stored in exactly one place. A relation that repeats a fact across many tuples can be split into smaller relations that each hold it once, joined back when a query needs them together, and the normal forms are the named conditions that say how far that decomposition has gone.

It shares a word and nothing else with [[Min-Max Scaling]], which rescales a numeric column so its training values land on a chosen interval. One transforms the values inside a single column and leaves the design of the database untouched; the other rearranges which relations exist and stores no value differently. Neither explains anything about the other, and a reader who arrived from feature preprocessing wants that note.

## Formal statement

### Functional dependency

Every normal form below is stated with one relation-level notion, so it is defined first. In a relation, a set of attributes $X$ **functionally determines** a set of attributes $Y$, written

$$X \to Y$$

when any two tuples that agree on $X$ also agree on $Y$. The dependency is a constraint on every legal state of the relation, not an observation about the tuples that happen to be there today. It is **trivial** when $Y \subseteq X$, since agreeing on $X$ then forces agreement on $Y$ for free.

Two derived terms are needed. A **candidate key** is a minimal set of attributes $K$ with $K \to$ all attributes of the relation, minimal meaning no proper subset of $K$ has that property; a relation can have several. A **superkey** is any set of attributes containing a candidate key. An attribute is **prime** when it belongs to at least one candidate key, and **non-prime** otherwise.

### First normal form

The traditional statement, and the one in the reading, has three parts: every column holds atomic values, with no sets, arrays or comma-separated lists inside a single cell; every row is uniquely identifiable, normally by a primary key; and related data goes into multiple rows rather than into repeated columns such as `phone1`, `phone2`, `phone3`.

That statement does not survive contact with its central word. "Atomic" is Codd's informal phrasing, and C. J. Date's objection ("What First Normal Form Really Means", chapter 8 of *Date on Database: Writings 2000-2006*, Apress, 2007) is the standard one: atomicity has no absolute meaning. A character string decomposes into substrings, a date decomposes into a year, a month and a day, and a fixed-point number decomposes into an integer part and a fractional part. If any of those counts as non-atomic then nothing is atomic; if none does, then a string holding `"red,green,blue"` is atomic and the condition has not ruled out the thing it was written to rule out.

The honest version is the one Codd's own formulation supports. What 1NF rules out is a **relation-valued attribute**: a domain whose elements are themselves relations, which in table terms is a column whose cells hold tables. Each cell holds exactly one value of its domain and nothing else. That is checkable from the schema alone, which is what makes it a condition.

What it therefore does not settle is the comma-separated list. A cell holding `"red,green,blue"` contains one string, and one string is one value of the string domain, so the relation is in 1NF on the formal condition. What is wrong with it is not a normal form violation but a modelling decision: the schema has declared the column to be text while the application reads it as a list, so the database can enforce nothing about the list and every consumer has to parse it identically or disagree silently. 1NF is the wrong tool for that argument, and reaching for it is where the atomicity wording sends people.

### Second normal form

A relation is in second normal form when it is in 1NF and has no **partial dependency**: no non-prime attribute is functionally determined by a proper subset of any candidate key.

Two things in that sentence are stronger than the reading's version, which says only that non-key attributes depend on the entire primary key. The condition quantifies over **every candidate key**, not merely the one a designer happened to designate as primary; the choice of primary key among the candidates is arbitrary, so a condition stated only over it would make normalization depend on an arbitrary choice. And the attributes it constrains are the **non-prime** attributes, those in no candidate key at all, not everything outside the primary key. Where candidate keys overlap, an attribute can sit outside the primary key and still be prime, and 2NF says nothing about it.

A relation in 1NF and not in 2NF:

```
OrderLine(order_id, product_id, quantity, product_name)
```

The only candidate key is $\{\texttt{order\_id}, \texttt{product\_id}\}$. The partial dependency is

$$\texttt{product\_id} \to \texttt{product\_name}$$

where $\{\texttt{product\_id}\}$ is a proper subset of that key and `product_name` is non-prime. So the product's name is repeated on every order line for that product, while `quantity`, which does depend on the whole key, is stored once per line as it should be. Splitting `product_name` out into a `Product(product_id, product_name)` relation removes the partial dependency and puts the name in one place.

### Beyond second normal form

The reading stops at 2NF. The two forms below are named here from research rather than from it, one line each, so that the ladder is not left looking two rungs tall, and neither gets a full treatment in this note.

- **Third normal form.** Additionally forbids a non-trivial dependency $X \to A$ where $A$ is non-prime and $X$ is not a superkey, which is the transitive case: a non-prime attribute determined by another non-prime attribute rather than by a key.
- **Boyce-Codd normal form.** Additionally forbids any non-trivial dependency $X \to Y$ where $X$ is not a superkey, dropping 3NF's exemption for prime attributes, so a determinant that is not a superkey is a violation whatever it determines.

Primary sources across the whole ladder: Codd's 1970 paper for the relational model itself, "A Relational Model of Data for Large Shared Data Banks", *Communications of the ACM* 13(6); Codd's "Further Normalization of the Data Base Relational Model", presented at the 1971 Courant Computer Science Symposium 6 and published in *Data Base Systems*, for the normal forms through 3NF; and for BCNF, E. F. Codd, "Recent Investigations into Relational Data Base Systems", *Proceedings of the IFIP Congress 1974*. Ian Heath had stated the same condition three years earlier, in "Unacceptable File Operations in a Relational Data Base", *Proceedings of the 1971 ACM SIGFIDET Workshop on Data Description, Access and Control*, which is why some writers call it Heath normal form.

### What normalization is buying

Less redundancy, and with it the three anomalies that redundancy causes. Each is concrete in the `OrderLine` relation above, before the split:

- **Insertion.** A new product that nobody has ordered yet cannot be recorded at all, because there is no row to put its name in until an order line exists.
- **Update.** Correcting a product's name means finding every order line that carries it, and one row missed leaves the database asserting two names for one product with no error anywhere.
- **Deletion.** Deleting the last order line for a discontinued product destroys the only record that the product ever had a name.

All three are the same fault seen from three directions: a fact about a product is being stored in a relation about order lines, so it can only be written, changed or destroyed alongside order lines.

### What it costs

Data ends up spread across relations, and a query wanting it together has to join them back. A join matches tuples across relations, so its cost scales with the sizes of the inputs rather than with the size of the answer, and on large relations that is expensive enough to shape how the query is written.

**Denormalization** is the deliberate reverse: reintroducing the redundancy, storing the product name on the order line again, so the read needs no join. The trade is exact and goes back the way it came. Reads get cheaper and every anomaly above comes back, so denormalization is defensible where writes are rare or where something else is responsible for keeping the copies in agreement, and indefensible as a default.

## Where it is used

[[Relational Model]] is where the vocabulary comes from. Normalization is stated over relations, attributes and dependencies among them, and it has no meaning in a document or graph store, where there is no relation to decompose.

[[Min-Max Scaling]] is the disambiguation link, held here because it claims the nearby word and means something unrelated.

[[Data Warehouse]] is the trade running the other way on purpose. A warehouse's dimensional schema is denormalized so that an analytical query touches few tables, accepting the redundancy because warehouse rows are loaded in bulk and then not updated, which is the condition under which the update anomaly cannot bite.

[[Online Transaction Processing]] is the workload normalization is for. Small writes to individual records are exactly where the anomalies show up, and exactly where storing each fact once is worth the joins on the read side.
