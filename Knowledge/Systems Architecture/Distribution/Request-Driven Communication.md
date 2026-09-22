---
note_kind: concept
aliases:
  - request-driven
  - request driven
  - request-driven architecture
  - request-response
  - REST
  - RESTful
  - REST API
  - RPC
  - remote procedure call
  - service-oriented architecture
  - SOA
up: "[[Distributed System]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

**Request-driven communication** is the mode of data flow in which one process sends a request across the network to another, stating what it needs, and the second returns the data over the same connection. The caller waits for that reply before it goes on, and the waiting is what makes the mode request-driven, rather than any particular protocol or message format.

The arrangement it is tightly coupled to is service-oriented architecture, which cuts a system into services that make requests of one another. The two ends can be two separate companies, in which case the request crosses an organizational boundary and the interface is effectively a published product, or two parts of one application, which is [[Microservices]]. That note carries the architectural contract for the second case and is not repeated here.

## Formal statement

The mode fixes four things about every call, and each one can be checked against a running system.

- **The caller names the callee.** It addresses a specific service at a specific endpoint, so the callee's identity sits in the caller's code or configuration. Changing who serves a request is a change to the caller or to something standing in for it.
- **The callee must be running at the moment of the call.** Nothing is holding the request on its behalf, so a callee that is down does not produce a delayed request, it produces no request at all.
- **The result comes back on the same connection**, inside the same call, rather than arriving later by some other route.
- **The caller therefore learns the outcome.** Success, an error, or a timeout: it gets one of the three, and it gets it at the point in its own code where the call was made.

The fourth clause is the one usually sold as the advantage, and it is a real one. The second is what decides how the mode behaves once systems are built out of it. Because an unavailable callee is an immediate failure for the caller rather than a delay, availability composes multiplicatively down a chain of synchronous calls. If serving one request needs $n$ services called in sequence, and service $i$ is independently available with probability $a_i$, the whole path is available with probability

$$A = \prod_{i=1}^{n} a_i$$

so the chain is never more available than its worst member and in general less available than any of them. Independence is an assumption and a generous one, since services sharing a datacenter, a deployment pipeline or a dependency fail together. This product is the sharpest checkable consequence of the mode, and it is exactly what the contract in [[Event-Driven Communication]] is chosen against: dropping the second clause is dropping the multiplication.

### REST and RPC

Two styles account for most of what is built in this mode, and neither earns a note of its own. What follows is what each one is and what separates them. How data is encoded on the wire, and how an interface changes without breaking callers who have not been updated, is the subject of chapter 5 of [[DDIA|Designing Data-Intensive Applications]] and belongs there rather than here.

**REST** was defined by Roy Fielding in his 2000 doctoral dissertation, *Architectural Styles and the Design of Network-based Software Architectures*. It is an architectural style, which is to say a named set of constraints a system either adopts or does not. It is not a protocol, not a message format and not a library, and describing it as something designed for requests over a network gets the category wrong. The constraints are what REST is, and Fielding states what each one buys:

- **Client-server**, separating the user interface from data storage, which lets the two evolve independently across organizational boundaries.
- **Stateless**, each request carrying everything needed to understand it, which improves visibility because a monitor need not look beyond a single request, reliability because recovering from partial failure is easier, and scalability because the server can free resources between requests.
- **Cache**, responses labelled cacheable or not, which can eliminate some interactions entirely.
- **Uniform interface**, which simplifies the overall architecture, improves the visibility of interactions and decouples implementations from the services they provide.
- **Layered system**, each component seeing only the layer it talks to, which bounds overall system complexity and lets proxies, caches and gateways be inserted without anyone else knowing.
- **Code on demand**, the one optional constraint, letting a server ship executable code to simplify the client.

Most things called REST APIs satisfy some of those and not the hypertext clause of the uniform interface, and the gap between the name and the practice is documented by the author of the term. Fielding wrote in October 2008 that he was, in his words, "getting frustrated by the number of people calling any HTTP-based interface a REST API", and that "if the engine of application state (and hence the API) is not being driven by hypertext, then it cannot be RESTful and cannot be a REST API. Period." He classifies those interfaces as RPC in style. So the phrase has two meanings in circulation: the style Fielding defined, and the ordinary industry convention of an HTTP interface with resource-shaped URLs and JSON bodies. The second covers most public APIs, and it is worth knowing which of the two a given claim about REST is about.

**RPC** is the other style. A remote procedure call makes a call to a service on another machine look, in the caller's source code, like calling a function or a method in the caller's own language. It is typically used between services owned by one organization, where both ends can be changed together and a shared interface definition is practical to maintain. [[Microservices]] names gRPC, the usual modern instance, with Protocol Buffers as its interface definition language and wire format.

The standing objection is that the resemblance is itself the problem. A remote call fails in ways a local call cannot: it can time out, arrive twice, or run to completion while the reply is lost, and that last case leaves the caller unable to tell "did not happen" from "happened, reply lost". Its cost is also different by orders of magnitude, which changes what is reasonable to put inside a loop. An abstraction that makes the call look local hides precisely the cases the caller has to handle.

**Why tRPC is not a third style.** Listing REST, RPC and tRPC together puts a library beside two architectural styles. tRPC is a TypeScript implementation of the RPC style, and its own documentation says as much, describing it as "one implementation of RPC, designed for TypeScript monorepos". It sits under RPC next to gRPC, not alongside RPC as a peer.

## Where it is used

[[Microservices]] is this mode raised to an architecture: every service exposes an API and every interaction between services is a call of the shape described above, which is why the availability product bites hardest there. [[Modes of Data Flow]] is where this sits among its alternatives, and its table is where the clauses above are lined up against the other two modes. [[Event-Driven Communication]] is the alternative whose contract is defined by dropping this one's second clause, so the two notes are best read against each other rather than separately.

[[Distributed System]] supplies the failure model every call here actually runs in, including the case where a crashed callee and a slow one are indistinguishable from the caller's side. [[Cloud-Native Architecture]] is assembled by building higher-level services out of lower-level cloud services that are reached over the network, so in such a system this mode is not one option among several but the way the system is put together at all.
