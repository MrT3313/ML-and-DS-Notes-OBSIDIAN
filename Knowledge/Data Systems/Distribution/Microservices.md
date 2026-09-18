---
note_kind: concept
aliases:
  - microservice
  - microservices
  - microservice architecture
  - microservices architecture
up: "[[Distributed System]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

**Microservices** is an architecture in which an application is split into services, each with one well-defined purpose, each exposing an API that clients and other servers call over the network, and each maintained by a single team.

It is usually described as a refinement of service-oriented architecture, which divides a system into clients and servers and lets clients make requests of the servers. That ancestry is the common account rather than a settled one. Fowler and Lewis note that SOA means too many different things, that what is met in practice under the name is often quite different, and that some microservice advocates reject the label outright while others regard microservices as, in their phrase, "service orientation done right".

The last property in the definition is the one doing the real work. Microservices is a technical solution to a people problem: it exists to let different teams make progress independently, without having to coordinate with each other on every change. Read that way, the architecture is a bet that the cost of network boundaries between components is lower than the cost of human coordination across a single codebase, and the bet only pays when the teams it is drawn around actually exist.

## Formal statement

The precise form is an architectural contract. A running system either honors each clause or it does not, and a system that claims the label while breaking a clause can be shown to be breaking it.

- **One well-defined purpose per service.** Each service does one job. There is no measure of how small that job must be, but there is a check: if you cannot say what the service is for in a sentence, this clause is not being honored.
- **The API is the only way in.** Clients and other servers reach the service over the network, through its interface and nothing else. No caller reads its files, its memory or its tables.
- **One team owns each service.** Ownership is exclusive. A service two teams both change is a service whose release schedule has to be negotiated, which is the coordination cost the architecture was adopted to remove.
- **Independent deployability is the property that follows.** Given the three clauses above, each service can be updated on its own schedule with no coordinated deployment of anything else, because clients see only the API and the owners can change what is behind it without affecting them. This is the property the whole architecture rests on, and it is the one to test first: if shipping one service requires shipping another at the same time, the arrangement is microservices in name only.
- **Its own database.** A strong preference rather than a defining clause. Fowler and Lewis list decentralized data management as one of nine characteristics of the style and say microservices prefer to let each service manage its own database, possibly on different database technologies. A system where two services share tables is still called microservices by most people who run one, but it has given up the property that makes independent release safe, since a schema change then crosses a team boundary.

### What the contract costs

- **Complexity.** Splitting a system into services breeds it, and the complexity moves from inside one program into the spaces between many.
- **Testing.** Exercising a path that crosses several services is much harder to arrange than calling a function.
- **Operations per service.** Each service needs infrastructure for deploying new releases, allocating hardware to match load, collecting logs and monitoring health, and that bill is multiplied by the number of services.
- **API evolution.** The interfaces are the contract between teams, and changing a contract that other teams already depend on is difficult, which is the whole reason interface description tooling exists.

## Where it is used

Microservices is one of the reasons a system becomes a [[Distributed System]] on purpose rather than by accident: the moment two services communicate over a network, every failure mode in that note applies to every call between them. [[Serverless]] is the same line pushed further, asking how fine a service can be cut and who pays for it while it is idle, and answering with metered billing for execution time instead of provisioned capacity.

The operational bill is what ties the architecture to the rest of the cloud material. [[Cloud-Native Architecture]] is the deployment style built around services of this shape, and [[DevOps]] is the organizational answer to the fact that each service now needs its own deployment, logging and monitoring, which is work that has to belong to the team that owns the service rather than to a separate department. That is the same people problem the architecture was adopted to solve, showing up again on the operations side.

### Tooling the per-service costs are paid with

- **Kubernetes.** An open source platform for managing containerized workloads, started at Google in 2014 and now maintained by the Cloud Native Computing Foundation, which handles deployment, restarting failed containers, scaling, service discovery and configuration, and is the usual answer to the per-service operations bill.
- **OpenAPI.** An open specification maintained by the OpenAPI Initiative for describing HTTP APIs in YAML or JSON, so an API's shape is a machine-readable artifact that documentation, client code and tests are generated from rather than agreed by conversation.
- **gRPC.** A remote procedure call framework, originally from Google and now a Cloud Native Computing Foundation project, in which a client calls a method on a server on another machine as if it were a local object, running over HTTP/2 with Protocol Buffers as its interface definition language and wire format, so the interface is a compiled artifact and an incompatible change is a build failure rather than a production surprise.

None of the three is described here beyond what it is for. Each is a large system with its own documentation, and this note is not a reference for any of them.
