---
note_kind: concept
aliases:
  - serverless
  - serverless computing
  - function as a service
  - functions as a service
  - FaaS
up: "[[Cloud Computing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Serverless, also called function as a service, means the cloud provider allocates and frees hardware resources automatically as incoming requests arrive. You upload code and a trigger; there is no instance to size, start or keep warm. There are still servers, of course. The name says only that none of them is yours to think about.

The point is the billing model, and it is a repeat of one that already happened. Cloud storage replaced capacity planning with metered billing: you stopped buying disks for the volume you expected and started paying for the bytes you actually stored. Serverless brings metered billing to code execution, so you pay for the time your code is running instead of provisioning resources in advance.

## Formal statement

The billing is where serverless is quantitative, and AWS Lambda is the reference implementation. Its charge has two components, a count of requests and a measure of resource-time, and the second is not time alone.

$$\text{cost} = N \cdot p_{\text{req}} + \left( \sum_{i=1}^{N} \frac{\lceil d_i \rceil_{1\text{ms}}}{1000} \cdot M \right) \cdot p_{\text{GB-s}}$$

where $N$ is the number of invocations, $d_i$ is the duration of invocation $i$ in milliseconds, $M$ is the memory allocated to the function in gigabytes, and $p_{\text{req}}$ and $p_{\text{GB-s}}$ are the per-request and per-gigabyte-second rates, which vary by region and processor architecture.

Three details of that expression carry the whole idea.

- **Duration is rounded up to the nearest 1 ms**, not to a coarse block. The unit is fine enough that a function doing almost nothing costs almost nothing.
- **Duration is multiplied by allocated memory**, giving gigabyte-seconds. You still make one sizing decision, the memory setting, and CPU is allocated in proportion to it, so a function is not free of capacity planning so much as reduced to a single dial.
- **Idle costs nothing.** With no requests there is no duration to bill, which is the whole difference from a provisioned instance.

The last point has an opt-out that is worth knowing, because it puts the old model back: provisioned concurrency charges for the amount of concurrency you configure and for the period you configure it, in addition to requests and duration. It is bought to avoid cold starts, and the price of avoiding them is paying in advance again.

So the raw phrasing, that you pay for the time your application is running, is approximately right and slightly under-specified. The unit is memory-time rather than time, and there is a per-invocation charge alongside it, so a workload of very many very short calls is priced quite differently from one long call.

## Where it is used

[[Cloud Computing]] is the arrangement this is the sharpest version of: metered billing has reached code execution, and the vendor now decides not only which machines run your code but whether any are running at all. [[Microservices]] sets the granularity question this answers, cutting a system into independently deployable services with one purpose each, and serverless takes the cut further, down to a single function with no long-running process behind it. That granularity is bought at a cost: every call between two functions is a network call, so a serverless application is a [[Distributed System]] with more edges than the same logic in one process would have had.
