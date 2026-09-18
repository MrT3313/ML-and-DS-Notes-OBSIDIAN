---
note_kind: concept
aliases:
  - HPC
  - high-performance computing
  - high performance computing
  - supercomputing
  - supercomputer
  - supercomputers
up: "[[Distributed System]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

**High-performance computing**, also called supercomputing, is the use of a large cluster for computationally intensive scientific tasks. The workload is one enormous calculation split across many machines, such as a climate model, a molecular simulation or a large training run, rather than a service answering requests from users.

## VS

The contrast worth drawing is against [[Cloud Computing]], since both are large clusters of machines in a building and they are built on opposite assumptions. Three properties separate them.

**The job is a restartable batch, and uptime matters less.** An HPC system runs large batch jobs submitted to a scheduler, and a job that fails can be stopped, fixed and restarted, so an interruption costs compute time rather than a broken promise to a user. In practice the restart is from a checkpoint rather than from the beginning: NERSC documents checkpoint and restart as the way to run a computation longer than the wall-clock limit of any single job, by splitting it into many shorter jobs chained together, and reports that long simulations get their best throughput that way. Restartability is therefore routine operating procedure, not just a failure story. A cloud service has no equivalent move, because stopping it is visible to everyone using it.

**Nodes talk over a high-speed interconnect, usually by remote direct memory access.** Shared memory and RDMA are easily run together here, and they sit at different levels. Within one node, the cores genuinely do share memory, which is what threads and OpenMP use. Between nodes there is no shared memory by default: they pass messages across a dedicated interconnect, with InfiniBand, Omni-Path, Slingshot and Ethernet the common ones according to NIST SP 800-223, and the message passing is typically implemented with RDMA, which writes directly into the remote machine's memory without involving its CPU or its kernel. Inter-node memory sharing does exist, listed by NIST SP 800-223 among the low-level tools vendors supply, but it is a product built on top of that interconnect rather than the default model. Either way the point stands: the interconnect is engineered for bandwidth and latency in a way that a cloud network is not, because a single tightly coupled calculation is waiting on every message.

**A high level of trust is assumed among users.** This is the price of the previous property, and it is explicit rather than accidental. NIST SP 800-223 records that direct memory access and communication between nodes may bypass the kernel, so the protections the kernel would have provided are lost, and that the only privilege boundaries separating users are often POSIX file system permissions, with root on one compute node potentially equivalent to root across the system. It also notes that HPC users tend to value security only so far as it does not slow the machine down. Cloud computing starts from the opposite premise, that tenants are mutually hostile strangers, and pays for hardware-level isolation accordingly.

## Formal statement

What separates high-performance computing from any other large cluster is a set of three conditions rather than a threshold, and each one is checkable against a given installation.

The job is a batch submitted to a scheduler and restartable from a checkpoint, so an interruption costs compute time and nothing that was promised to a user. Memory is shared within a node and not between nodes, and everything that crosses a node boundary crosses a dedicated interconnect, usually by remote direct memory access that writes into a peer's memory without involving its CPU or its kernel. Users are assumed to trust one another, far enough that kernel protections are deliberately bypassed and the privilege boundary between two users is often no more than file system permissions.

All three together make an HPC system, and each can be held on its own, which is what makes the in-between cases worth naming: a cloud with a fast interconnect is still a cloud while its tenants stay mutually hostile, and a trusted cluster running long-lived services is not an HPC system however fast its network is.

**Not further quantitative at this depth.** The term names a kind of installation and a way of using it, so there is no quantity it denotes; the numbers that get quoted belong to one particular machine rather than to the category.

## Where it is used

[[Cloud Computing]] is the comparison this note exists to draw, and the two differ on what happens when a machine goes away, on what the network is optimized for, and on whether the other users are trusted. [[Distributed System]] is the general case both belong to, and HPC is a useful reminder that not every reason for distributing work is about serving users: here the reason is that one calculation is simply too large for one machine, which is the scalability entry in that note and none of the others.

[[Node]] means the same thing here as anywhere, a participant with its own state that reaches the others by messages, but the assumptions attached to it differ sharply. An HPC compute node assumes its peers are cooperative and its interconnect is fast and private, and an algorithm written against those assumptions does not survive being moved onto a network where neither holds. [[Separation of Storage and Compute]] is the other visible difference: many HPC compute nodes have no local disk at all and read from a shared parallel file system, which is the same disaggregation arriving from a different direction.
