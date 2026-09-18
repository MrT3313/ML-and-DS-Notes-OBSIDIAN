---
note_kind: concept
aliases:
  - distributed system
  - distributed systems
  - distributed
  - single-node system
  - single node system
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A **distributed system** is a system that involves several machines communicating via a network. Each process taking part in it is a [[Node]].

The definition is short on purpose, and everything that makes such systems hard follows from what it leaves out. The nodes share no memory and no clock, so nothing one node knows is available to another until it is put in a message and that message arrives. The network between them can delay, drop, duplicate and reorder those messages, and any node can stop at any moment, so a silent peer and a dead peer look identical from outside. What holds the arrangement together is only the protocol the nodes agree to run, and the goal of that protocol is usually to make the collection behave, from a user's point of view, like one service.

## VS

The contrast is the **single-node system**: one machine running one process, where state lives in shared memory, where a function call either returns or the whole program dies with it, and where there is no partial failure to reason about because there is no network in the middle. Every property in the definition above is a cost paid to get out of that situation, so the question is never "is distributed better" but "is the reason strong enough".

It is frequently not, and the sharpest published version of that is the COST result of McSherry, Isard and Murray (HotOS 2015). They defined COST as the Configuration that Outperforms a Single Thread, the hardware a platform needs before it beats a competent single-threaded implementation, and measured it on graph workloads against a single thread on a 2014 laptop. Their conclusion was that many published systems have a COST of hundreds of cores, and several have no configuration at all that beats one thread. The measurements are in the formal statement below, and they are the reason this comparison is worth making at all rather than a matter of taste.

## Formal statement

Two things can be stated exactly here, and neither is a derivation.

The first is the system model, the set of assumptions every distributed algorithm is proved against. For $n$ nodes: no shared memory, no shared clock, communication only by messages that may be delayed, dropped, duplicated or reordered, any node liable to crash at any point, and a crashed node indistinguishable from a slow one. There is no formula in that and it needs none, because it is exact and checkable as it stands. An algorithm is either proved against these assumptions alone or it has quietly helped itself to a stronger one, and finding which is the first thing to ask of any protocol.

The second is the measured price of distribution, from the COST paper above. Twenty PageRank iterations over the twitter_rv graph took 275 seconds in a single thread, against 249 seconds for GraphLab, 419 for GraphX and 857 for Spark, each on 128 cores. Label propagation over the same graph took 153 seconds in a single thread against 200 seconds for Giraph, 242 for GraphLab and 1784 for Spark, all at 128 cores, and swapping the algorithm for union-find brought the single thread down to 15 seconds. The ratio is the part to carry: on label propagation, Spark on 128 cores was more than eleven times slower than one thread on one laptop, so a cluster of over a hundred cores losing to one computer is not a hypothetical.

## Where it is used

A distributed system is not usually a design goal, it is the consequence of one of the reasons below, and it drags the costs below along with it whether or not they were wanted. [[Node]] is the unit it is built from and the thing every count in it refers to. [[Microservices]] is the standard way of cutting an application into one deliberately, [[Cloud Computing]] is the deployment model that makes most systems distributed almost by accident, and [[High-Performance Computing]] is the other tradition entirely, distributing a single calculation rather than a service.

### Why a system ends up distributed

- **Inherent distribution.** If an application involves two or more interacting users, each using their own device, then the system is unavoidably distributed, because communication between the devices has to occur over a network.
- **Requests between cloud services.** When data stored in one service is processed in another, that data must be transferred over the network, which is why [[Cloud-Native Architecture]] and [[Microservices]] are distributed by construction.
- **Fault tolerance and high availability.** The application needs to continue working even if one machine, or several, goes down.
- **Scalability.** When data volume or computing requirements grow bigger than a single machine can handle, the load can be spread across multiple machines.
- **Latency.** With users around the world, you may want servers in several regions, so that packets do not have to travel across the world for every request.
- **Elasticity.** An application that is busy at some times and idle at others can scale up and down with demand in a cloud deployment, which is hard on a single machine that has to be provisioned for its maximum load.
- **Specialized hardware.** Different parts of the system can run on different types of hardware, matched to their workload.
- **Legal compliance.** Data residency laws can require that particular data physically sit in a particular country. Russia's Federal Law No. 242-FZ, in force since September 2015, is a genuine residency rule: personal data of Russian citizens must be recorded, systematized, accumulated, stored, updated and retrieved using databases located in Russia. It is worth separating from transfer rules, which are a different obligation. GDPR Chapter V, articles 44 to 50, restricts sending personal data to a third country unless conditions such as an adequacy decision or appropriate safeguards are met, but it nowhere requires the data to be stored inside the EU.
- **Sustainability.** Flexibility about where and when jobs run allows them to be scheduled at times or in places with plenty of renewable energy available.

### What it costs

- **Every network request has to deal with the possibility of failure.** In a single process a call returns or the process dies; across a network it can also time out, arrive twice, or succeed while the reply is lost, and each of those is a case the calling code now owns.
- **It is hard to troubleshoot.** No single machine holds the whole story of what happened, and reconstructing one from several partial accounts, recorded against clocks that do not agree, is its own skill.
- **It is not optimized for all workloads.** In some cases a single-threaded program on one computer outperforms a cluster of over 100 CPU cores, as the COST measurements above show, because the partitioning, serialization and coordination a cluster needs are pure overhead that the single machine never pays.
