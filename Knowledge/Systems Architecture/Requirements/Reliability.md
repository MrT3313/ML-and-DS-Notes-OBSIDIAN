---
note_kind: concept
aliases:
  - reliability
  - system reliability
  - reliable system
up: "[[Systems Architecture]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

Reliability is the requirement that the system continue to perform the correct function at the desired level of performance even in the face of adversity, and adversity means three things at once: hardware faults, software faults, and human error. It is a property of the whole system seen from outside it, which is what separates it from any statement about a component: what is being asked is not whether every part is healthy but whether the user is still getting the service while some part is not.

## Formal statement

The requirement turns on a distinction that ordinary use of the word "failure" loses.

- A **fault** is one component of the system deviating from its specification. A disk stops answering, a process exhausts its memory, a packet is dropped, an operator applies a configuration that was meant for somewhere else.
- A **failure** is the system as a whole stopping to provide the required service to its user.

A fault is an internal event and a failure is an external one, and the entire engineering content of reliability lives in the gap between them. **Fault tolerance** is the property of anticipating faults and coping with them so that they do not turn into failures. That makes fault tolerance a means and reliability the end, which is why the two are not interchangeable words: a system can be fault tolerant against the faults it was designed for and unreliable in production because the faults it met were other ones.

What a fault class asks for differs by class, and redundancy is not a general answer.

| fault class | one instance of it | what tolerating it takes |
|---|---|---|
| hardware | a disk dies, a power supply fails, a rack loses its network | redundancy across units that fail independently, which is why physical separation is part of the claim rather than a detail of it |
| software | a bug triggered by one unusual input, a process leaking memory until it is killed, one slow service stalling every caller that waits on it | redundancy does not touch it, because every replica runs the same code and meets the same input. What helps is isolation between components, limits that stop one component consuming everything, and the ability to put the previous version back |
| human | a misconfiguration applied to production, a destructive command run against the wrong environment | paths that make the mistake undoable, an environment to make it in first, and checks that catch the change before it reaches anything a user touches |

**No system tolerates every possible fault.** Tolerating everything would mean surviving the loss of the region, the vendor and the planet, and nothing is built to that. So the requirement is always relative to a stated fault class, and a claim of the form "this system is reliable" that does not name the class of faults it survives cannot be found false against anything and is therefore not a claim. The checkable form names three things: which faults, what the system still does while one of them is present, and the measure that "still does" is read on, an error rate, a latency percentile, or a probability of loss over a stated period.

"Correct function" is itself relative to a specification, so reliability is not correctness. A system that implements the wrong specification exactly, and goes on implementing it while machines fail around it, is reliable and wrong at the same time. Reliability asks whether the promised service survives adversity, never whether the promise was the right one to make.

### Why the fault classes are rates rather than events

At scale a fault class stops being something that might happen and becomes something that happens at a rate. For $N$ independent units with an annual failure rate $r$,

$$\mathbb{E}[\text{failures per year}] = N \cdot r$$

Schroeder and Gibson measured $r$ for disks over more than 100,000 drives in production (FAST 2007). Datasheet mean time to failure was 1,000,000 to 1,500,000 hours, which implies a nominal annual replacement rate of at most 0.88%, while observed replacement rates typically exceeded 1%, with 2% to 4% common and up to 13% at some sites. Taking $N = 10{,}000$ and $r = 0.04$ gives 400 replacements a year, more than one a day. A design that treats a dead disk as an exceptional event is wrong by arithmetic at that size, not by taste.

The human class is the one designed against least and fired most. Oppenheimer, Ganapathi and Patterson studied failure data from three large internet services (USITS 2003) and found operator error the largest single cause of failure in two of the three, with more than half of the operator errors that caused a failure being configuration errors, and nearly all of them in one of the services. Operator errors also took longer to repair than the other classes. This is the argument for counting the deployment path, the staging environment and the rollback as part of the reliability design rather than as conveniences.

## Where it is used

[[Distributed System]] is where faults are the standing case rather than the exception, because its system model already grants that a crashed node and a slow one are indistinguishable from outside, so every request has to be written as though the fault has already happened. [[Object Storage]] is what a checkable claim looks like in a product: it publishes a designed annual durability figure and names the fault class that buys it, the loss of an availability zone, which is exactly the pairing the formal statement above asks for. [[DevOps]] is who carries the obligation once the system is running, since monitoring, on-call and learning from incidents are how a fault is caught before it becomes a failure and how the next one is made cheaper.

[[Machine Learning Systems Design]] is the specialization where this requirement is hardest to check, because a learned system can fail it while every component keeps answering and every health check stays green, and [[Model Rot]] is that failure under its own name.
