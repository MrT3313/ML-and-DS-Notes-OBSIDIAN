---
note_kind: concept
aliases:
  - node
  - nodes
up: "[[Distributed System]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A **node** is one participant in a [[Distributed System]]: a single process that holds its own state, runs on its own, and reaches the other participants only by sending messages across a network.

The word is often used loosely to mean "a machine", and in most deployments one machine does run one node, which makes the two readings agree in practice. They are not the same claim, and the process reading is the one to keep. Van Steen and Tanenbaum define a distributed system as a collection of autonomous computing elements, and say that such an element, generally called a node, can be either a hardware device or a software process. What makes something a node is the role it plays in the protocol, not the box it is running in.

The difference shows up as soon as the two readings disagree. Three database nodes running on one laptop for a test are three nodes, not one, because each keeps its own state and they talk to each other by messages. A machine running twenty shards is running twenty participants. Going the other way, a node's storage need not be inside the machine at all, since a virtual disk is a service on other machines that is made to behave like a local one.

## Formal statement

What can be stated exactly about a node is the test for being one, and it has four parts. A node holds private state that no other participant can read directly. It keeps its own clock, which no other participant shares. It is reachable only by messages, never by a reach into its memory. And it fails on its own, so its stopping does not stop the others and theirs does not stop it. Anything with all four is a node, whether it is a machine, a virtual machine, a container or one of twenty processes on a laptop, and anything missing one of them is not.

The node is also the unit the system model of a [[Distributed System]] counts in. A replication factor, a quorum size and a failure threshold are all counts of nodes, and none of them is well defined until the participant, and not the box it runs in, is the thing being counted.

**Not further quantitative at this depth.** A node is a unit of a system model, so it is what other quantities are counted in rather than a quantity of its own.

## Where it is used

A node is defined only relative to the thing it takes part in, so [[Distributed System]] is where the word gets its meaning and where the reasons for having more than one are set out. It is also the unit the failure argument in that note is about, since a network request between two nodes is exactly the request that can fail.

[[Separation of Storage and Compute]] is the case that makes the process reading pay off. A local disk dies with the instance it is attached to, while a virtual disk can be detached from one instance and attached to another, so the storage outlives the node that was using it. That sentence is only sayable if a node is a participant with a lifetime, rather than a piece of hardware. [[High-Performance Computing]] uses the same word for its compute nodes, with a different set of assumptions about what they may do to each other.
