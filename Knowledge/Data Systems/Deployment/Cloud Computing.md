---
note_kind: concept
aliases:
  - cloud
  - cloud computing
  - cloud service
  - cloud services
  - public cloud
  - managed service
  - managed services
  - software as a service
  - SaaS
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Cloud computing is paying a vendor to run machines and software on your behalf and reaching the result over a network. The vendor owns the hardware, employs the people who keep it running, and bills for what you consume rather than for capacity you bought in advance.

It is a separate decision from the older one of whether to build a piece of software or buy it. Between building and buying sits a middle ground: off-the-shelf software, open source or commercial, that you deploy yourself. Once you know *what* software you are running, *how* you deploy it is still open, and the two answers are cloud or on premises. Either answer to the first question is compatible with either answer to the second.

## VS

The choice against [[Self-Hosting]] has no general winner. Two conditions do most of the deciding.

- **Whether you already know how to operate this particular system.** If you have experience setting the system up and running it, buying your own machines and running the software on them is often cheaper. If you do not, adopting a cloud service is usually easier and quicker than learning to manage the system yourself, and adoption becomes a configuration step rather than a project. The provider is serving many customers' workloads on the same software, so the operational expertise is amortized across all of them instead of being rebuilt inside each one.
- **Whether your load is predictable.** Predictable load can be sized once and bought once. Load that varies a lot over time is the case where a cloud service earns its premium, because you scale up and down as needed instead of owning peak-load infrastructure that sits unused most of the time.

Everything else in the comparison follows from who holds the machines, and what it costs is control. Five concrete consequences:

| situation | what you can actually do about it |
|---|---|
| the service lacks a feature you need | politely ask the vendor, who may or may not add it, at some unknown time in the future |
| the service goes down | wait for it to come back up, with no access to the failing machines |
| something behaves wrongly | debug from the outside, using only the signals the vendor chooses to expose |
| the vendor raises prices or shuts the service down | you are at their mercy, which is what vendor lock-in means in practice: leaving costs whatever the migration costs |
| the data is regulated | the vendor has to be trusted to keep it secure, and that trust has to be made legible to whoever enforces the privacy and security rules you are under |

[[Self-Hosting]] states the same trade from the other side.

### What it cost in 2022

Two published figures, both from 2022, give the argument a number.

| comparison | figure |
|---|---|
| bare machine rented from a dedicated-server provider | 128 physical cores, 512 GB of memory, 50 Gbps, at $1,318 per month |
| nearest on-demand cloud instance | AWS m6a.metal, 96 physical cores, 768 GB, 50 Gbps, at $8.2944 per hour, about $6,055 per month |
| buying the equivalent machine outright | roughly $40,000 for a comparable 128-core, 512 GB Dell, which pays back in about 8 months against the cloud instance and about 30 months against the rental |

That comparison is Nima Badizadegan, "Use One Big Server", specbranch.com, August 2022. The same piece prices the equivalent workload on AWS Lambda at roughly $46 per hour, about 5.5x the cloud instance and about 25x the rental, which is what the finest-grained metering costs when the load is steady rather than spiky.

What that ratio looks like as an annual line item: David Heinemeier Hansson reported in October 2022 that his company was paying Amazon over half a million dollars a year for managed database and search alone, RDS and Elasticsearch, and left the cloud over it ("Why We're Leaving the Cloud", world.hey.com, October 2022).

Both sources are written by people arguing for leaving, and both quote 2022 list prices that have moved since. Treat the ratio as the durable part and the absolute numbers as dated. The figures also only price hardware: staff time, outage risk and regulatory exposure share no unit with dollars per month, which is why the decision has a contract and a price range rather than a formula.

## Formal statement

The vendor owns the machines, employs the operators, decides what is exposed and what is not, and sets the price. You own the configuration, the bill, and the cost of leaving. Every row of the control table above is that one line applied to a situation, and each row is checkable against a named service: either the status page is the only signal during an outage or it is not, either the export path exists or it does not.

## Where it is used

[[Self-Hosting]] is the other side of the deployment question, and the two notes are only meaningful together. [[Cloud-Native Architecture]] is what a system looks like when it is designed for this arrangement from the ground up rather than lifted onto it, and [[Object Storage]] and [[Serverless]] are the two services that show most clearly what designing for it buys. [[DevOps]] is who does the operating once a vendor runs the machines: the work does not disappear, it changes shape, with capacity planning becoming financial planning and performance optimization becoming cost optimization. Anything assembled out of cloud services is a [[Distributed System]] whether you meant it to be or not, because a request from one service to another crosses a network and can fail on the way.
