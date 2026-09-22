---
note_kind: concept
aliases:
  - row-major
  - row major
  - row-major order
  - column-major
  - column major
  - column-major order
  - row-oriented
  - column-oriented
  - columnar storage
  - columnar
  - row storage
  - storage layout
  - memory layout
up: "[[Data Serialization]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Row-major and column-major are the two ways of flattening a two-dimensional table into the one-dimensional sequence that memory and files actually are. Under **row-major** order the consecutive elements of a row sit next to each other, so the table is stored one whole record after another, which is what CSV does. Under **column-major** order the consecutive elements of a column sit next to each other, so the table is stored one whole column after another, which is what Parquet does. Nothing about the table changes: same values, same rows, same columns, different order on the way out.

## Formal statement

Take a table of $m$ rows and $n$ columns. Element $(i, j)$, with $0 \le i < m$ and $0 \le j < n$, lies at offset

$$\text{row-major: } \; in + j \qquad\qquad \text{column-major: } \; jm + i$$

from the start. Both layouts hold the same $mn$ elements and differ only in which direction of travel is contiguous. Stepping to the next column costs a stride of $1$ in row-major order and $m$ in column-major order; stepping to the next row costs $n$ in row-major order and $1$ in column-major order. Every claim below follows from those two strides.

### Reading a few columns

A query that wants $k$ of the $n$ columns, over all $m$ rows, touches $mk$ elements under a column-major layout: those columns are $k$ contiguous runs and nothing else has to be read at all. Under a row-major layout it touches $mn$, because the wanted values are interleaved with every other field of every record, so the reader takes whole rows and throws most of each away. The ratio of what has to be read is

$$\frac{mk}{mn} = \frac{k}{n}$$

and the point of it is what is missing: there is no $m$. The saving is set entirely by the fraction of columns wanted, and it is the same for a thousand rows as for a trillion. Growing the table does not change the advantage, it changes what the advantage is worth.

With numbers in it: a table of thousands of rows and thousands of columns, of which four columns are wanted. At $n = 1000$ and $k = 4$ the ratio is $4/1000 = 0.004$, four tenths of one percent of the table against all of it. Reading the same four columns row-major means reading all thousand and filtering down afterwards. AWS states the identical arithmetic for Amazon Redshift's columnar blocks: a table of 100 columns queried on 5 of them reads about five percent of the data, where a row-wise store would read the blocks holding the other 95 as well.

### Reading whole rows

The advantage reverses, and not in the way an element count alone suggests. Reading $r$ whole rows touches $rn$ elements either way. Row-major, those $rn$ elements are $r$ contiguous runs, one per record. Column-major, they are $r$ elements taken from each of $n$ separately located column segments, so the same count arrives from $n$ discontiguous places instead of $r$. The element counts are equal; the locality is not, and locality is what costs.

This is also why writes favour row order. Appending one record is a single write at a single place in a row-major layout and $n$ writes into $n$ places in a column-major one. Column stores answer by batching rather than by streaming columns the length of the table: Parquet cuts the table into row groups and stores one column chunk per column inside each group, which bounds how much has to be buffered before anything can be written and keeps the footer's offsets finite.

### Why contiguous beats strided

That modern computers process sequential data more efficiently than nonsequential data is usually asserted and rarely explained. The mechanism is the cache line. Memory moves between RAM and cache in fixed-size lines, 64 bytes on current x86-64, and a read of one element fetches the whole line containing it. For an element of size $s$ bytes, a contiguous sweep therefore gets $L/s$ elements per fetch and pays

$$\frac{s}{L} \text{ fetches per element, against } 1 \text{ for a stride of at least } L$$

so the worst case ratio in memory traffic between a strided walk and a contiguous one is $L/s$, which is $8$ for 8-byte doubles on a 64-byte line: seven eighths of every line fetched is discarded. On top of that, the hardware prefetchers described in the Intel 64 and IA-32 architectures optimization reference manual detect a sequential stream of accesses to adjacent lines and fetch the next ones before the program asks, which a strided walk across a row-major table does not trigger. The same argument repeats one level down at coarser granularity, where the unit is a disk block (Redshift uses 1 MB) or a ranged request to an object store rather than a cache line.

| | row-major | column-major |
| --- | --- | --- |
| contiguous direction | along a record | along a field |
| elements read for $k$ of $n$ columns | $mn$ | $mk$, a fraction $k/n$ |
| elements read for $r$ whole rows | $rn$, in $r$ runs | $rn$, from $n$ scattered segments |
| one appended record | one write | $n$ writes, so batched into groups |
| typical instance | CSV, an OLTP table | Parquet, an analytical store |

In a machine learning table the rows are [[Training Instance|instances]] and the columns are [[Feature|features]], which is what turns the ratio above into a rule of thumb: a row-major format is the one to want when whole instances are being read, and a column-major format when a few features are being pulled out of many.

### The layout in NumPy and pandas

The claims below are about implementations, so they are true of a version. NumPy is checked against the current stable documentation, and pandas against 3.0.6, the release current in September 2026.

NumPy lets you choose. The `order` argument of an array constructor is documented as "row-major (C-style) or column-major (Fortran-style) order", `'C'` is the default, and `'F'` gives the Fortran layout. So a design matrix $\mathbf{X}$ built the ordinary way has its rows contiguous: iterating instances sweeps memory, and pulling out one feature column strides by $n$.

pandas is column-oriented, but not by being one column-major array. A DataFrame's values live in a `BlockManager`, which the implementation describes as managing "a bunch of labeled 2D mixed-type ndarrays": columns sharing a dtype are consolidated into one block whose first axis is the columns, so within a block each column's values are contiguous and a frame of three integer columns and two float columns holds two such arrays rather than one. Since pandas 2.0 a column can instead be Arrow-backed, one Arrow array per column, selected with `dtype_backend="pyarrow"` or an `ArrowDtype`, and pandas 3.0 makes the default string dtype PyArrow-backed where pyarrow is installed. Whichever of those a given column is, the invariant that matters holds: a column is one run and a row is not, so `df["a"]` reads a run and `df.iloc[0]` gathers one element from each column's storage and, across mixed dtypes, has to build something new to put them in.

## Where it is used

[[Online Transaction Processing]] reads and writes individual records by key, which is exactly the access a row-major layout makes contiguous, so the storage preference that workload carries is not a convention but the $k/n$ arithmetic above read in the direction where $k = n$. [[Online Analytical Processing]] is the reverse: it names columnar storage as the property its downstream design choices follow from without saying what columnar storage is, and the ratio here is the reason a scan of a few columns over a huge number of records is affordable at all. [[Hybrid Transactional-Analytical Processing]] is the two layouts inside one product, row storage in TiKV serving the transactional side and column storage in TiFlash serving the analytical one, with the planner choosing per query.

[[Data Serialization]] is where this sits among the other choices a format makes: layout is the access-pattern axis, the one that decides whether a reader can get at one field without consuming everything before it. [[Data Lake]] is where both orders turn up side by side, since a lake enforces no format: Avro is the row-oriented option and Parquet the column-oriented one, and which of them a file is decides what a query over it costs.
