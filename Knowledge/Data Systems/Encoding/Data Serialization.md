---
note_kind: concept
aliases:
  - data serialization
  - serialization
  - serialisation
  - deserialization
  - serialization format
  - serialization formats
  - data format
  - data formats
  - file format
  - file formats
  - text format
  - binary format
up: "[[Data Systems]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Data serialization is converting a data structure or an object's state into a form that can be stored or transmitted and reconstructed later. Deserialization is the return trip. A *serialization format* is the agreed rule for making that trip, and picking one is picking three things at once: whether a person can read the result, whether the bytes are text or binary, and what a reader has to do to get at one piece of what is in there.

Text and binary are the coarse split. A text file is one whose bytes are meant to be read as characters in some encoding, which usually means a person can open it and see what it says. Binary is the catchall for everything else: files whose bytes stand for numbers, lengths and tags directly, written to be read by a program rather than by a person. Binary is the more compact of the two for anything but tiny values, and it is not so much unreadable as unreadable without the decoder.

## Formal statement

A format fixes two functions over the data model it supports: an *encoding* that maps a value to a sequence of bytes, and a *decoding* that maps bytes back to a value. Round-tripping is well defined exactly when

$$\text{decode}(\text{encode}(x)) = x \qquad \text{for every } x \in D$$

where $D$ is the set of values the format's data model admits. Both halves carry weight. A value outside $D$ has no representation at all, which is why a format is only ever as good as its data model: JSON has one number type and no date type, and CSV has no types whatsoever, every field being text that the reader has to interpret. And a format that decodes to something other than what was encoded round-trips lossily, which is what happens when a float is written out in decimal with too few digits.

Everything else the term covers sits on three axes, each of which can be settled for a given format by inspection rather than by reputation.

| axis | the question it answers | how to settle it |
| --- | --- | --- |
| human readability | can a person read the bytes without running a decoder | open a file in a text editor and see whether the values are there |
| text against binary | is every byte part of a character in some encoding, or may a byte take any of its 256 values | look for a character encoding in the specification. RFC 8259 requires JSON text to be UTF-8 on the open network; the Parquet specification opens and closes a file with the four-byte marker `PAR1` |
| access pattern | to reach one field, must the reader consume the stream from the beginning, or can it jump straight to it | look for length prefixes, offsets or an index. JSON and CSV have none and are read front to back; Parquet's footer carries the byte offset of every column chunk, so a reader fetches the footer first and then only the chunks it wants |

The size consequence of the second axis is arithmetic rather than folklore. A non-negative integer $v \ge 1$ written as decimal text costs $\lfloor \log_{10} v \rfloor + 1$ bytes and grows with the value, while a fixed-width binary integer costs 4 or 8 bytes whatever the value is. So $1234567890$ is 10 bytes of text against 4 as a 32-bit integer, and $7$ is 1 byte of text against the same 4. Text wins on small values and loses on everything else, and it loses again on repetition: a JSON document repeats every field name in every record, where a binary format with a schema sends a field number or nothing at all.

What a format does **not** fix, unless its specification says so:

- **The schema.** Which fields exist, what their types are and which are required is a separate agreement. JSON, CSV, BSON and pickle impose none. Protocol Buffers and Avro require one, and it lives outside the encoded bytes.
- **Forward and backward compatibility.** Whether a new reader can read old data, and an old reader new data, comes from following the format's evolution rules (Avro's schema resolution, the stability of Protocol Buffers field numbers), not from having chosen the format.
- **Whether the writer's schema travels with the data.** Avro's object container file stores it in the header as JSON, and Parquet's footer carries it, so both are self-describing. A bare Protocol Buffers message is not: the documentation states that a message does not describe itself and cannot be fully interpreted without its `.proto` file. A CSV header row is a convention that names columns and says nothing about their types.

## Where it is used

[[Row-Major and Column-Major Order]] is the access-pattern axis stated in full, and it is the axis with the largest consequences: it decides whether reading a few columns costs a fraction of the file or all of it.

[[Data Lake]] fixes no format at all, which is the whole point of it, so the files in one are whatever the producers happened to write. [[Data Warehouse]] is the opposite arrangement: data is conformed to a schema on the way in, and after that the storage format is the engine's private business rather than a file anyone chose. Amazon Redshift documents exactly this, converting loaded records into its own columnar blocks of 1 MB.

[[Object Storage]] is where these files usually sit, and its interface decides which formats are comfortable there. An object store has no partial write and no general append, so a format meant to be edited in place fits badly, while one written whole and read back by ranged request fits well. That is the pairing behind Parquet on S3: the reader issues one ranged read for the footer and then one per column chunk it actually wants.

### The formats that turn up most

Each of these is one instance of the contract above, not a topic covered here.

- **JSON** is a text format for structured data, specified by RFC 8259, which is an Internet Standard (STD 90) and describes it as "a lightweight, text-based, language-independent data interchange format". ECMA-404 describes the same syntax, and the two bodies have committed to keeping the documents aligned, so they are two statements of one format rather than two formats. Human readable, self-describing in field names, schemaless, and read front to back.
- **CSV** is the plain text table: one record per line, fields separated by commas. RFC 4180 is Informational, not a standard, and says so of itself, that it documents the format in use and that "there is no formal specification in existence, which allows for a wide variety of interpretations of CSV files". So a claim that a CSV file *is* anything is a claim about the producer, not about the format. Row-oriented, untyped, no index into it.
- **Parquet** is a binary column-oriented file format. The table is cut into row groups, each row group into one column chunk per column, and the metadata is written last, in a footer, so a file can be written in a single pass and read by consulting the footer first.
- **Avro** is a binary row-oriented format with a required schema, written in JSON. The specification defines two encodings, a binary one and a JSON one, so "binary" is the usual choice rather than the only one. Its object container file carries the writer's schema in the header, which is what lets a program that did not write the file read it.
- **Protocol Buffers** is Google's "language-neutral, platform-neutral extensible mechanism for serializing structured data": a `.proto` schema compiled into code, and a compact binary wire format that does not describe itself.
- **Pickle** is CPython's own object serialization. Binary by default, Python specific by design, and able to reconstruct object graphs that JSON cannot represent, at a price the maintainers state plainly: the module is not secure, and it is possible to construct malicious pickle data that executes arbitrary code during unpickling, so data from an untrusted or tampered source must not be unpickled.
- **BSON** is "a binary-encoded serialization of JSON-like documents", designed to be traversable, and it is the primary data representation of MongoDB. It is what the document model reaches for when JSON text is too slow or too loose about types.

Formats outlive the systems they were introduced with, so what reads them is worth stating for today rather than for the year the format appeared. Parquet is the data file format of Apache Spark and of the lakehouse table formats: the Apache Iceberg specification defines table management over "immutable file formats: Parquet, Avro, and ORC", and Amazon Redshift reads Parquet in S3 through Spectrum external tables rather than storing it internally. Avro is alive in two other places: as the format Iceberg writes its own manifests in, and as a message encoding on Kafka, where Confluent's Schema Registry describes it as "the original default format" alongside JSON Schema and Protocol Buffers. Protocol Buffers is what TensorFlow puts inside a TFRecord file, which is itself only "a simple format for storing a sequence of binary records" and holds a serialized `tf.train.Example` message by convention rather than by requirement. Pickle is what PyTorch's `torch.save` and `torch.load` are built on, which is why `torch.load` carries the pickle warning and why its `weights_only` argument, restricting the unpickler to tensors and primitive types, became the default in PyTorch 2.6.
