---
note_kind: concept
aliases:
  - cloud native
  - cloud-native
  - cloud native architecture
  - cloud-native system
  - cloud native systems
up: "[[Cloud Computing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A cloud-native system is one designed for the cloud from the start rather than moved onto it. The thing it is defined against is a system built to run on a machine in your own rack and then run on a rented machine instead, which changes who owns the hardware and nothing else.

What changes when you design for the cloud is that the system stops treating a machine as the unit it lives on. It builds on the cloud's own lower-level services instead of only on the resources the operating system hands it, it spreads across machines it never names, and it grows and shrinks with load.

A list of technologies is not a definition, and the closest thing to an authoritative definition is careful about this. CNCF's Cloud Native Definition v1.1, approved in February 2024, characterizes cloud native by a property: loosely coupled systems that interoperate in a manner that is secure, resilient, manageable, sustainable and observable, developed and deployed at scale in a programmatic and repeatable way. Only after that does it mention technologies, and it hedges them twice: cloud-native architectures "typically consist of some combination of" containers, service meshes, multi-tenancy, microservices, immutable infrastructure, serverless and declarative APIs, and it states outright that the list is non-exhaustive. Version 1.0 was blunter about the logical role of that list, saying those technologies "exemplify this approach".

So containers, [[Microservices]], [[Serverless]], dynamic scaling and managed services are common ingredients, not the thing itself.

In principle any software you can self-host could also be offered as a cloud service. The claim worth examining is that systems built for the cloud from the ground up can do things the lifted ones cannot.

## Formal statement

The term names a property rather than a quantity, so the precise form is a pair of tests that a running system either passes or fails.

**The structural test.** A cloud-native service uses not only the computing resources its operating system manages, but lower-level cloud services, to build higher-level ones out of them. [[Object Storage]] is the standing example of a lower level. A cloud-native analytical database does not manage disks; it puts its files in an object store and runs query workers that read from it, which is [[Separation of Storage and Compute]] and is the defining structural property of the whole category. Put as something that can come back false: storage capacity and processing capacity can be resized independently of each other, and the durable state survives the loss of every compute node currently holding it. A system where growing the dataset means provisioning a larger machine fails this test, however it is deployed and whoever owns the rack.

**The property test.** Membership is not decided by the technology list. A system running containers, microservices, serverless functions, dynamic scaling and managed services, while remaining tightly coupled and unobservable, does not qualify. A system using none of the five while being loosely coupled, resilient, manageable and observable does. Both halves are checkable against a named system, which is what makes this a criterion rather than a slogan.

Past those two the term is a stance, and the advantages claimed for the stance have been measured only for particular products. What those measurements actually say is recorded under [[#What the claimed advantages actually rest on]], because a result about one database is not a statement about the category.

## Where it is used

[[Separation of Storage and Compute]] is the structural property that makes a system cloud-native rather than merely cloud-hosted, and it is what the whole category is organized around. [[Object Storage]] is the lower-level service the higher-level ones are layered on, which is why the same handful of object stores sit underneath so many analytical products. [[Cloud Computing]] is the deployment arrangement this is designed for, and [[Self-Hosting]] is what a cloud-native system gives up: the right-hand column of the table below is exactly the set of systems nobody but the vendor can operate, so choosing one is choosing the loss of control that note describes. [[Microservices]] and [[Serverless]] are two of the ingredients CNCF lists as typical, and are the granularity at which such systems tend to be cut.

### What the claimed advantages actually rest on

Four advantages are claimed for ground-up cloud-native design: better performance on the same hardware, faster recovery from failures, quicker scaling of compute to match load, and support for larger datasets. Every one of them traces back to a paper written by the engineers who built the system being measured, published in an industry track, with no released configurations and no independent replication. They are worth taking seriously and they are not established facts.

| claim | what actually stands behind it |
|---|---|
| better performance on the same hardware | Aurora's designers (Verbitski et al., SIGMOD 2017, all authors at Amazon Web Services) report up to 5x MySQL throughput on SysBench and 2.3x to 16.3x on a TPC-C variant, against a MySQL baseline they configured and did not publish, with the headline 35x figure measured against a deliberately expensive synchronously cross-zone-mirrored MySQL. The one independent controlled study, Pang and Wang at Purdue (SIGMOD 2024), rebuilt the architecture on PostgreSQL to isolate each design principle and found the reverse at the architectural level: on fixed hardware, disaggregating storage alone costs 16.4x on reads and 17.9x on writes, which caching and the log-as-the-database trick reduce but do not erase |
| faster recovery from failures | Aurora's paper gives one parenthetical, recovery "generally under 10 seconds", with no experiment, no distribution and no baseline. Spanner (Corbett et al., OSDI 2012, all authors at Google) does have a real measurement: after a hard kill of the leader zone, throughput recovers in about 10 seconds, bounded by the Paxos leader lease. It is measured against Spanner's own failure injection, not against another system |
| quicker scaling of compute to match load | architecturally the most plausible of the four and the least measured. Neither paper runs an elasticity experiment. Aurora measures throughput at different fixed instance sizes, which is scalability rather than rescaling time |
| support for larger datasets | a product limit rather than a finding. Aurora documents a 64 TB volume ceiling; Spanner states a *design target* of trillions of rows, while the production dataset it actually reports is tens of terabytes, which its own authors call small compared to many NoSQL deployments |

The honest summary is that these are the designers' claims about their own systems, and that the only vendor-neutral measurement anyone has published points the other way on the first one.

### Systems on each side of the line

The split below is by what you can do with the product, not by what it is good at. The left column is software you can download and run on machines you control; the right column is software nobody outside the vendor can run at all.

| category | can be self-hosted | available only as a service |
|---|---|---|
| operational, [[Online Transaction Processing]] | MySQL, PostgreSQL, MongoDB | Amazon Aurora, Azure SQL Database Hyperscale, Spanner |
| analytical, [[Online Analytical Processing]] | ClickHouse, Apache Spark, Teradata | Snowflake, BigQuery, Azure Synapse Analytics |

What each one is, and where the naming has drifted:

- **MySQL** (Oracle) and **PostgreSQL** (the PostgreSQL Global Development Group) are relational databases you install. PostgreSQL has no first-party cloud offering at all, which makes it the cleanest example in the table.
- **MongoDB** (MongoDB, Inc.) and **ClickHouse** (ClickHouse, Inc.) are a document database and a columnar analytical database, both downloadable and both also sold as managed services by their own makers, Atlas and ClickHouse Cloud. "Self-hosted" here means *can be* self-hosted, not *only*.
- **Apache Spark** (Apache Software Foundation) is a processing engine, not a data store. It executes computation over storage that lives elsewhere, so it belongs in an analytical list but it is a different kind of thing from the databases beside it.
- **Teradata** is a company, not a product. Its analytical platform was Teradata Vantage and was renamed the Teradata Autonomous Knowledge Platform in 2026. It deploys both on premises and in the cloud, so its place in the self-hosted column is only half right.
- **Amazon Aurora** (Amazon Web Services), not "AWS Aurora", is a MySQL- and PostgreSQL-compatible database whose storage layer is a separate distributed service. It cannot be run outside AWS.
- **Azure SQL Database Hyperscale** (Microsoft) is a *service tier* of Azure SQL Database rather than a product of its own, which makes listing it beside Aurora and Spanner as a peer slightly generous.
- **Spanner** (Google Cloud), not "Google Cloud Spanner", is a globally distributed relational database. Google no longer positions it as purely relational, describing it now as relational, graph, key-value and search. A local emulator exists for development and Google states it is not for production.
- **Snowflake** (Snowflake Inc.) is an analytical database, and it is cloud-native in a different sense from the other three in its row: those are first-party services of the cloud that runs them, while Snowflake owns no datacenters and runs as third-party software on AWS, Azure and Google Cloud. Its documentation is explicit that it cannot be installed locally or on private infrastructure.
- **BigQuery** (Google Cloud), not "Google BigQuery", is a serverless analytical warehouse where query compute is allocated per query rather than provisioned.
- **Azure Synapse Analytics** (Microsoft) still exists and is still supported, with no announced retirement date, but it has been superseded for new work by Microsoft Fabric, which dropped the Synapse name from every one of its workloads and ships a first-party migration path off Synapse dedicated SQL pools. One component, Synapse Data Explorer, was retired in October 2025.

Product lists go stale faster than anything else in a note, and four of these twelve names were already wrong or imprecise when written down.
