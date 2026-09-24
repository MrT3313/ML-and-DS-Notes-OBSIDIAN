---
note_kind: concept
aliases:
  - scalability
  - scalable
  - horizontal scaling
  - vertical scaling
up: "[[Systems Architecture]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## Definition

Scalability is a system's ability to cope with growth, where coping means holding the performance it promised while some number describing its load goes up. Growth comes in more than one form: the thing being run can get bigger, as when a model that fits in a gigabyte of memory is replaced by one with a hundred million parameters, and the traffic can get heavier, as when a service answering ten thousand requests a day starts answering between one and ten million. The two are different problems with different answers, which is the first reason the word cannot be used on its own.

## Formal statement

Scalability is not a one-dimensional label that a system either has or lacks. It is a question, asked about one particular kind of growth: if the system grows in this way, what are our options for coping with it. Saying "this system is scalable" is therefore meaningless shorthand, and saying "this system cannot scale" is the same shorthand with a minus sign.

Making the question precise takes two definitions.

- A **load parameter** is the number that describes the load, and which number it is depends on the system. Requests per second for a web service, the ratio of reads to writes on a database, the count of simultaneously active users, the hit rate on a cache. The right one is the one the bottleneck actually responds to, and picking it is part of the design work rather than a preliminary to it.
- A **performance measure** is what "coping" is read on: throughput for a batch system, response time for an online one, and for response time a percentile rather than a mean, since the mean is held down by the majority of fast requests and says nothing about the tail that users notice.

With both named, two questions make a scalability claim checkable:

1. Increase a load parameter and keep the system resources unchanged. How is the performance measure affected?
2. Increase a load parameter. How much do you have to increase the resources by to hold the performance measure where it was?

A claim that names no load parameter and no performance measure cannot be found false against either question, and so is not a claim at all. The checkable form is a sentence with numbers in every slot: at ten times the current requests per second, on four times the machines, the 99th percentile response time stays under 200 milliseconds.

### Two directions to add resources

The second question has two answers and they are not interchangeable.

| | vertical scaling (scaling up) | horizontal scaling (scaling out) |
|---|---|---|
| what you add | a bigger machine: more cores, more memory, faster disks | more machines, each of ordinary size |
| ceiling | the largest machine anyone builds and sells, and the price per unit of capacity climbs as you approach it | no ceiling of that kind |
| what it asks of the design | nothing. The program does not have to know it happened | the work has to be divisible and the state has to be placed, which makes the system a [[Distributed System]] and imports every cost of being one |
| what happens when one unit is lost | the service is down | the rest carry on, if the design allows it |

Only one of the two is bounded by what a single machine can be built to be, which is why the two are not substitutes even at the sizes where both would work. The usual arrangement is neither in its pure form: scale up while it is the cheap answer, and cut the system so it can scale out before the ceiling is the thing deciding.

### What growth in the thing being run costs

When the thing being run gets bigger, the resources one copy of it needs go up with it, and that sets a floor for every machine that has to hold a copy. The floor is arithmetic. For $P$ parameters at $b$ bytes each with $k$ copies of each parameter held at once,

$$M_{\text{params}} = P \cdot b \cdot k$$

Serving holds one copy of the weights, $k = 1$. Training with Adam holds four, the weights, the gradients and two moment estimates, so $k = 4$. Mixed precision arrives at the same total from a different direction: Rajbhandari, Rasley, Ruwase and He count 16 bytes per parameter for mixed-precision Adam (ZeRO, SC 2020), 2 for the half-precision weights, 2 for the half-precision gradients and 12 for the single-precision master copy plus the two moments. [[Model Quantization]] is a change to $b$ itself, so an 8-bit copy of the weights, $b = 1$, needs a quarter of the single-precision floor.

So for $P = 10^8$ at $b = 4$: serving is 0.4 GB and training is 1.6 GB. Neither is 16 GB. The rest of any training figure is activation memory, the intermediate values of the forward pass held so the backward pass can use them, and that term scales with the batch size, the input length and the depth rather than with $P$. Sixteen gigabytes is reachable for a model of this size, but only as a training figure and only once a batch size is stated with it. Quoted with no batch size it is not a figure about anything, which is the same defect as a scalability claim with no load parameter, arriving one level down.

### What growth in traffic costs

Traffic is the load parameter that gets quoted per day and has to be paid per second. For $N$ requests a day the mean rate is

$$\lambda = \frac{N}{86400} \text{ requests per second}$$

so 10,000 a day is about 0.12 per second and 10 million a day is about 116. The mean is the wrong number twice over. It hides the peak, since traffic that follows waking hours arrives at several times the daily average inside the busy hour, and it hides fan-out, since one user request can be many internal ones and the internal count is what the machines see. Capacity has to cover the peak of the busy period and the bill is paid on the whole range, which is why a load that fluctuates between one and ten million a day, a factor of ten between its quiet day and its busy one, is a different problem from a steady ten million and not a smaller one.

## Where it is used

[[Distributed System]] is the standard answer once one machine is not enough, and it already counts scalability among the reasons a system ends up distributed on purpose, alongside the costs that answer drags in with it. [[Separation of Storage and Compute]] is what lets the two halves answer the second question on their own schedules, so a query that has grown can be given more compute without anybody buying storage. [[Serverless]] and [[Cloud Computing]] are where that second question gets answered with money instead of hardware: capacity is rented by the hour or by the millisecond, so the response to a load parameter that moves is a bill that moves with it rather than a purchase sized for the peak.

[[Machine Learning Systems Design]] is where the axes a learned system adds are counted, beyond the two above.
