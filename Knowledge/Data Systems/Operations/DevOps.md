---
note_kind: concept
aliases:
  - DevOps
  - dev ops
  - devops
  - development and operations
  - operations engineering
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

DevOps is the practice of integrating the roles of software development and operations into teams with a shared responsibility for both backend services and the data infrastructure underneath them. The name is a contraction of *development* and *operations*, and it dates from the first devopsdays, held in Ghent in 2009 and founded by Patrick Debois. The talk usually treated as the practice's founding example is John Allspaw and Paul Hammond's account, at Velocity 2009, of deploying Flickr more than ten times a day with the two groups cooperating rather than negotiating.

Operations is the job the practice organizes, and it has two halves. One is ensuring services are reliably delivered to users, which covers configuring infrastructure and deploying applications. The other is ensuring a stable production environment, which covers monitoring and diagnosing any problems that may affect reliability. The sharp part is what the job is now about: the emphasis has shifted from individual machines to services, so the question being asked is no longer whether a particular box is healthy, but whether the service that box participates in is still answering.

Traditionally that job belonged to roles kept apart from the people writing the code, *database administrators* (DBAs) and *system administrators* (sysadmins). Merging them into one team brings a set of practices with it:

- setting up automation, so that a repeatable process replaces manual one-off jobs;
- using ephemeral virtual machines and services rather than long-running servers;
- enabling frequent application updates;
- learning from incidents;
- preserving the organization's knowledge about the system even as individual people come and go.

The last two are about people rather than machinery, and they are why the thing is described as a culture rather than a toolchain. A team can buy every tool on the list and still lose what it knew when the person who knew it leaves.

*Site reliability engineering* is the nearby discipline and is not another name for this one. Google, where SRE originated in 2003, frames the relation both ways in the same breath: DevOps can be read as a generalization of several core SRE principles to a wider range of organizations, or SRE as one specific implementation of DevOps with idiosyncratic extensions of its own. Either reading makes SRE the narrower and more prescriptive of the two, an operations team staffed by software engineers and held to explicit availability targets, so it is a neighbour rather than a synonym.

[[MLOps]] is this same practice carried out on a system whose behaviour is learned from data rather than written by hand, and the difference is not cosmetic. An ordinary service fails when something breaks. A deployed model degrades while every component keeps working correctly, because the world it was fitted on moved out from under it, which is [[Model Rot]]. So MLOps adds concerns that have no counterpart here: watching the input distribution rather than only the error rate, choosing a retraining cadence and the signal that triggers it, and keeping the previous model available to roll back to when a freshly trained one turns out worse in production. Everything in this note still applies underneath all of that; none of it is replaced.

### Where the operations burden moves in a cloud deployment

When somebody else runs the machines, the role splits in two rather than disappearing. An infrastructure company's operations team specializes in the details of providing reliable service to a large number of customers, and the whole point of paying for that is that those customers spend as little time and effort as possible on infrastructure. But customers of a cloud service still require operations. It is simply focused on different aspects, and two substitutions carry most of it: capacity planning becomes financial planning, and performance optimization becomes cost optimization. Both are the same move. A question that used to be answered by ordering hardware is now answered by reading a bill. That framing of the split is the one taken from [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]].

The customer-side column has since been named and organized as *FinOps*, a Linux Foundation project since 2020, which defines itself as an operational framework and cultural practice for maximizing the business value of technology through collaboration between engineering, finance, and business teams. The name follows the same pattern as DevOps and so does the structure: a wall between two groups removed by giving them shared responsibility, with finance standing where operations stood.

## Formal statement

**Not further quantitative at this depth.**

DevOps names a division of responsibility and the practices that follow from it, not a quantity, so no formula belongs to it. That division is set out in full above, and it is the entire content of the term.

## Where it is used

[[Cloud Computing]] is where the operations burden moves when a vendor runs the machines, and [[Self-Hosting]] is where it stays when nobody else does, which makes the choice between them partly a choice about which kind of operations staff you intend to employ. [[Serverless]] is the extreme form of the same substitution: capacity planning disappears entirely into a bill, because there is no capacity left for the customer to plan.

[[Microservices]] is where the cost shows up in an architecture. Every service needs its own deployment, its own logging, and its own monitoring, and that operational tax is exactly what the architecture buys independent deployability with, which is why the trade only pays off for an organization that already has the practices above in place.

[[MLOps]] is this practice carried onto systems whose behaviour is learned, and [[Model Rot]] is the failure mode driving it, one that ordinary operations has no equivalent of.

When the practice is studied empirically rather than argued about, what gets measured is the delivery performance of the teams practising it. Google Cloud's DORA program currently reports five such metrics, change lead time, deployment frequency, failed deployment recovery time, change fail rate, and deployment rework rate, grouped into throughput and instability. Each is an outcome the practice claims to improve, so they are the usual evidence offered for it rather than a measurement of it.
