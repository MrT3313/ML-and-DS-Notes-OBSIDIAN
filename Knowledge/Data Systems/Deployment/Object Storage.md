---
note_kind: concept
aliases:
  - object storage
  - object store
  - object stores
  - blob storage
  - S3
  - Amazon S3
up: "[[Cloud Computing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

An object store keeps whole files, each under a name, in a container reached over the network, and offers little beyond writing one and reading one back. It gives up most of what a filesystem can do. In exchange it hides the physical machines: the service spreads the data across many of them by itself, so running out of disk space on any one machine is not something you can do, and a machine or a disk failing does not lose data.

Amazon S3, Azure Blob Storage and Cloudflare R2 are three instances of the category. S3 came first and its API became the de facto interface, to the point that R2 is documented as implementing the S3 API so that applications can move without being rewritten.

## Formal statement

The interface is deliberately narrower than the one it replaces, and the contract is which filesystem operations survive the translation and which do not. The rows below are Amazon S3 against POSIX filesystem semantics; the other object stores differ in detail rather than in kind.

| operation | a POSIX filesystem | an object store |
|---|---|---|
| write part of an existing file | seek to an offset and write those bytes | not available. A write replaces the whole object. S3 states that it never adds partial objects, and that updating even one piece of metadata means putting the entire object again |
| append | open for append and write | not a general operation. Azure Blob Storage carves out a separate blob type for it, the append blob, which is an admission that the ordinary one cannot |
| rename or move | one call, atomic within a directory | on general purpose S3 buckets there is no rename: you copy to the new key and delete the old one, which is two operations and is not atomic. A single-call atomic `RenameObject` exists only for S3 directory buckets on the Express One Zone storage class, added in 2025 |
| directories | a real tree, with `mkdir`, `rmdir` and atomic directory operations | on general purpose S3 buckets the keyspace is flat. `photos/puppy.jpg` is one key and the slash is just a character in it, so what looks like a folder is a shared prefix and nothing more. S3 directory buckets are the exception and do organize objects into hierarchical directories |
| read part of a file | seek and read | available, as a ranged read |
| consistency | defined by the local filesystem | S3 gives strong read-after-write consistency for writes and deletes of objects in all regions, and updates to a single key are atomic. There is no locking for concurrent writers, so simultaneous writes to one key resolve last-writer-wins, and there is no way to update two keys atomically |

What the narrowness pays for is durability you do not administer, and that is the one quantity the interface does guarantee. S3 Standard is designed for 99.999999999% durability of objects over a given year, eleven nines, so the expected annual loss per object is

$$P(\text{loss}) = 1 - 0.99999999999 = 10^{-11} \text{ per object-year}$$

which puts a ten-million-object bucket at an expected $10^{-4}$ lost objects a year, one object every ten thousand years. The figure is bought by storing each object redundantly across a minimum of three Availability Zones, physically separated data centers within one region, and neither the disks nor the replication is visible in the API. The only thing you size is the bill.

## Where it is used

[[Separation of Storage and Compute]] is this idea in its plainest form: an object store stores and does not compute, so anything you want to know about the data has to be worked out somewhere else. A [[Data Lake]] is usually built directly on one, because raw files of every shape are exactly what a flat keyspace with cheap bytes is good at holding, and because a lake defers structure to query time rather than imposing it on write. [[Cloud-Native Architecture]] treats an object store as the lower-level service that higher-level ones are layered on, which is why the same three products appear underneath so many analytical systems. Reaching one is a network call that can fail, so an application built on object storage is a [[Distributed System]].
