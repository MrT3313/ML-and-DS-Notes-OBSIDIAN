---
note_kind: concept
aliases:
  - data engineering
  - data engineer
  - data engineers
  - analytics engineer
  - analytics engineers
  - analytics engineering
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Data engineering is the work of connecting the two kinds of system an organization runs and of owning the infrastructure that connection is made of. It is a field defined by its roles more than by any single technique, and there are two of them.

A *data engineer* is someone who knows how to integrate the operational and analytical systems and who takes responsibility for the organization's data infrastructure. That is close to the whole job description. The two systems are built for opposite access patterns and are normally separate databases, so somebody has to own the movement of data between them, along with the pipelines, schedules, and storage that movement runs on.

An *analytics engineer* models and transforms data to make it more useful for business analysts and data scientists in an organization. The title is recent and traceable: practitioners in the analytics community started using it in late 2018 and early 2019, Michael Kaminsky's "The Analytics Engineer" (January 2019) is the first piece written about it, and dbt Labs adopted it and did most of the work of spreading it. Those accounts add one thing to the description above that is worth keeping, namely what makes the role *engineering*: software engineering discipline is applied to the analytics code itself, so transformations are version controlled, tested, documented, and deployed, rather than being SQL passed between people.

### The teams a data engineer sits between

On the operational side are backend engineers, who build and run the services and databases that record what users do, an [[Online Transaction Processing]] workload. On the analytical side are business analysts (the business intelligence function) and data scientists, whose work runs against an [[Online Analytical Processing]] system.

The asymmetry between the two sides is what creates the role. Analysts and data scientists perform analytics and typically do not modify the data; they read it. What they do produce is [[Derived Data]], datasets in which the original data has been processed in some way, which are new artifacts sitting beside the original rather than edits to it. So the flow a data engineer maintains runs mostly one way, out of the [[System of Record]] and into somewhere the same facts can be read at length without degrading the service the users are waiting on, and the analysts' own output accumulates alongside that rather than flowing back into it.

## Formal statement

The division of labour set out above is a description of how the work splits once there is enough of it, not a rule, and the boundaries are not sharp in practice. One person holds several of these roles in a small organization, and the analytics engineer in particular is often an analyst who drifted toward engineering rather than a separate hire.

**Not further quantitative at this depth.**

These are roles rather than measurable quantities, so the division of labour is the whole of what can be stated precisely.

## Where it is used

[[Online Transaction Processing]] and [[Online Analytical Processing]] are the two systems the role exists to connect, and the reason the role exists at all is that no single database serves both workloads well. [[Extract-Transform-Load]] is the machinery that connection is usually made of, so it is the data engineer's standing deliverable rather than an occasional job.

[[Data Warehouse]] is the thing an analytics engineer shapes data for, a store whose schema is chosen for the questions analysts ask rather than for the transactions that produced the rows, and [[Data Lake]] is the looser alternative that accepts raw files first and leaves the modelling until later. [[Data Silo]] is what the organization gets when nobody holds the role: data trapped in the system that produced it, reachable only by the team that owns that system.

[[Derived Data]] is the output side of the same picture, what an analyst produces when they process an original instead of modifying it, which is why read access to the [[System of Record]] is usually enough for the analytical side.
