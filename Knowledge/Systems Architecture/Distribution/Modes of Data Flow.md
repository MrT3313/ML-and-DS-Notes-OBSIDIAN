---
note_kind: concept
aliases:
  - modes of data flow
  - mode of data flow
  - dataflow
  - data flow
  - dataflow mode
  - dataflow modes
up: "[[Distributed System]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

Two processes that do not share memory cannot pass a value by handing over a pointer, so how data gets from one to the other is a decision somebody has to make. A **mode of data flow** is one of the three standard answers, and what tells the three apart is a single question: what sits between the two processes.

Data can pass through a store, where one process writes and the other reads afterwards. It can pass through a request, where one process calls the other over the network and waits for the reply, which is [[Request-Driven Communication]]. Or it can pass through a broker, which accepts a broadcast from one process and holds it for whoever has arranged to be interested, which is [[Event-Driven Communication]]. Those two have notes of their own; the store mode is carried here in full.

One rule of thumb is worth keeping, stated as a rule of thumb and not as a property of either mode. The request mode suits logic-heavy work, where one process needs a specific answer computed on demand before it can continue. The event mode suits data-heavy work, where a large volume has to reach consumers whose identities are not the producer's concern.

## Formal statement

Each mode fixes an answer to the same five questions, and every cell below is a claim a running system either honors or breaks.

| mode | what sits between | does the producer name the consumer | must the consumer be running when the data is sent | where the data waits | what the producer learns |
|---|---|---|---|---|---|
| through a store | a database, table, file or object both processes can reach | no, it names a location | no | in the store, until something overwrites or deletes it | that the write committed, and nothing about who read it |
| through a request | nothing holds the data; a connection carries it | yes, it addresses a specific service | yes | nowhere, so an absent consumer is a failed call | the outcome of the call and the result itself |
| through a broker | a topic or queue held by a message broker | no, it names a topic or a queue | no, within the broker's retention window | in the broker, for a bounded configured period | that the broker accepted the event, and nothing about consumption |

Three consequences fall straight out of the table. The store and the broker columns are separated in time, so producing and consuming need not overlap, while the request column is not, which is what makes the callee's availability the caller's problem. Only the request column returns any information about the consumer to the producer, so it is the only one of the three in which a broken consumer is visible at the point where the data was sent. And only the store column leaves the data in place indefinitely by default, which is why it is the one that raises a question of authority rather than of delivery.

### Passing data through a store

Process A writes to a database and process B reads what A wrote. Nothing addresses anybody: A knows the table, B knows the table, and neither needs to know the other exists. Two requirements follow, and both are real constraints rather than restatements.

The first is reach. Both processes must be able to access the same database. That replaces a coupling between the two processes with a coupling of both to one store, which is why the mode is comfortable inside one organization and awkward across two: granting an outside process access to your database is granting it far more than the one exchange calls for.

The second is cost. Every value passed crosses the store twice, written once and read once, and reading and writing through a database can be slow enough that this mode is unsuitable for an application with a strict latency requirement. The store is on the critical path of every exchange, and unlike a network hop it is doing durable work each time.

What the mode buys in return is the whole of the second column: the reader can be down for as long as it likes, the data is still there when it comes back, and any number of readers can take the same rows without the writer doing anything differently.

## Where it is used

[[Request-Driven Communication]] and [[Event-Driven Communication]] are the two modes with notes of their own, and the table above is where the choice between them is framed before either note is read. [[Distributed System]] is the condition that makes the question unavoidable rather than optional: no shared memory and no shared clock is exactly the situation in which a value has to be put into a message, a row or an event before anyone else can see it. [[Microservices]] turns the choice into a standing one, since every boundary the architecture draws is a place where one of the three modes gets picked, and picking a different one at a different boundary in the same system is normal.

On the store mode specifically, [[Extract-Transform-Load]] is that mode running on a schedule: its stages hand data to one another by writing to storage rather than by calling one another, which is precisely what lets a stage fail and be rerun without the stages around it being rerun too. [[System of Record]] is the question the store mode forces, because once several processes read and write the same store, which of them holds authority over a given field stops being obvious and has to be assigned rather than discovered.
