---
note_kind: concept
aliases:
  - edge computing
  - edge device
  - edge devices
  - mobile edge computing
  - multi-access edge computing
  - MEC
up: "[[Systems Architecture]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## Definition

Edge computing is running the computation at or near the place where the data is produced and the answer is used, on the device itself or on infrastructure a short network hop away, instead of shipping the data to a distant data centre and waiting for the result to come back. Its opposite pole is computation done in the cloud, public or private, where the device only collects input and displays output.

The edge is a range rather than a point. Shi, Cao, Zhang, Li and Xu ("Edge Computing: Vision and Challenges", *IEEE Internet of Things Journal* 3(5), October 2016) count as edge any computing and network resource sitting on the path between the data sources and the cloud data centre: a phone is the edge between the sensors on a body and the cloud, a home gateway is the edge between the appliances in a house and the cloud, and a small data centre placed near a mobile network is the edge between a phone and the cloud. The same paper treats fog computing as the same idea seen from the infrastructure side. The far end of that range, servers operated by a network carrier at its base stations or aggregation points, has its own standard: ETSI's industry group for it was called Mobile Edge Computing and was renamed Multi-access Edge Computing (MEC) in March 2017 to take in Wi-Fi and fixed-line access as well as cellular.

An **edge device** is any machine at that near end which does the work itself: browsers, phones, laptops, smartwatches, cars, security cameras, robots and embedded controllers. FPGAs and ASICs belong on a different list. They are kinds of chip, a reprogrammable one and a single-purpose one, and they turn up *inside* edge devices (a camera, a car or a phone may carry one to do the heavy arithmetic) as well as inside cloud servers, so naming one says what the computation runs on and not where.

This note answers a different question from [[Cloud Computing]]. That note is about who owns and operates the machines; this one is about where the computation physically runs relative to the data. The two are independent. A vendor can ship, update and bill for code that runs on hardware in your pocket, which is edge placement with vendor operation, and a server you rent bare and administer yourself in a colocation facility a continent away is [[Self-Hosting]] with no edge placement at all. NIST's definition of a private cloud makes the same point from the other direction: the infrastructure may be owned by the organization or a third party, and may sit on or off premises (NIST SP 800-145, September 2011).

## VS

Against computing in a data centre, whether rented as [[Cloud Computing]] or run as [[Self-Hosting]], four things change.

- **Cost.** Cloud compute is expensive, and [[Cloud Computing]] carries the dated 2022 figures that put the nearest on-demand instance at about $\$6{,}055 / \$1{,}318 \approx 4.6$ times the monthly price of a rented bare machine. Work moved onto a device the user already bought does not show up on the operator's bill at all. It is not free, though: it moves onto the device's processor, memory and battery, which are smaller and shared with everything else the device does. Whether leaving the cloud is worth doing for cost alone is argued separately, in chapter 10 of DMLS.
- **Availability without a network.** A request served from a data centre needs a working connection for the whole round trip; a computation on the device needs none. Using the product rule from [[Request-Driven Communication]], a cloud-served answer is available with probability $a_{\text{link}} \cdot a_{\text{service}}$, where $a_{\text{link}}$ is the probability the device has a usable connection at that moment, while an on-device answer is available with probability $a_{\text{device}}$ alone, with no link term in it. Where connections are absent or unreliable, $a_{\text{link}}$ is the smallest factor and the edge is the only arrangement that works at all.
- **Latency.** Every cloud-served request pays a network round trip on top of its compute, and that round trip has a physical floor that no engineering removes. The derivation and the condition under which the edge wins are in the formal statement below.
- **Data custody.** Sending data to a data centre exposes it twice: in transit, where it can be intercepted, and at rest, where it sits alongside the data of every other user and makes one attractive target. Keeping raw data on the device shrinks both exposures, because less is transferred and less is stored centrally.

The regulatory side of custody needs stating carefully. The EU's General Data Protection Regulation (Regulation (EU) 2016/679) requires that personal data be limited to what is necessary for the purposes of processing (Article 5(1)(c), data minimisation), and requires the controller to build measures implementing that principle into the design of the processing and to ensure that by default only the personal data necessary for each purpose is processed, a duty it applies explicitly to the amount collected, the extent of processing, the period of storage and who can access it (Article 25(1) and (2), data protection by design and by default). Processing on the device and sending back only a result is one way to satisfy those duties, so it makes compliance easier to show. It does not make a system compliant by itself: whatever does leave the device, and whatever the vendor does with the device's data, still needs a lawful basis, a stated purpose and appropriate security, and a system that ships raw data home "just in case" gets no credit for having an edge component.

## Formal statement

Split the time a request takes into three terms: the network round trip $T_{\text{net}}$, time spent waiting in queues $T_{\text{queue}}$, and time spent computing $T_{\text{comp}}$,

$$T = T_{\text{net}} + T_{\text{queue}} + T_{\text{comp}}.$$

The first two belong to this domain. The third, when the computation is a model producing a prediction, is inference latency and belongs to [[Model Inference]], which uses the floor derived here.

### The network floor

Signals in optical fibre travel at the speed of light divided by the fibre's group index of refraction, $v = c / n_g$. With $c = 299\,792\,458$ m/s exactly (the SI definition) and $n_g \approx 1.468$ for standard single-mode fibre (Corning's datasheet for SMF-28 Ultra gives 1.4676 at 1310 nm and 1.4682 at 1550 nm, September 2019),

$$v = \frac{2.998 \times 10^8\ \text{m/s}}{1.468} \approx 2.04 \times 10^8\ \text{m/s},$$

which is about $4.9\ \mu\text{s}$ per kilometre one way. ITU-T Recommendation G.114 (May 2003), Table A.1, rounds this to a planning value of $5\ \mu\text{s/km}$ for optical fibre systems, allowing for repeaters and regenerators, and marks it provisional. A request to a server at fibre distance $d$ has to go there and come back, so

$$T_{\text{net}} \;\ge\; \frac{2d}{v} \;\approx\; \frac{2d}{2 \times 10^8\ \text{m/s}}.$$

For $d = 100$ km that is $2 \times 10^5\ \text{m} \,/\, (2 \times 10^8\ \text{m/s}) = 10^{-3}$ s, so the floor is about **1 ms of round trip per 100 km of one-way distance**. It is a floor in three senses: $d$ is the length of the fibre actually laid, which is never shorter than the straight-line distance and usually longer; it counts propagation only, before any router, queue or retransmission; and it ignores the device's own access link (Wi-Fi, a cellular radio), which adds its own delay before the signal reaches fibre at all.

### When the use case is impossible

A use case with a latency budget $L$ cannot be served from a data centre whose nearest usable region sits at distance $d_{\min}$ whenever the floor alone exceeds the budget,

$$\frac{2 d_{\min}}{v} > L \quad\Longleftrightarrow\quad d_{\min} > \frac{vL}{2}.$$

With a 10 ms budget, $vL/2 = (2 \times 10^8)(10^{-2})/2 = 10^6$ m, so any region more than about 1,000 km of fibre away is ruled out before a single instruction of the computation runs, and anything closer only leaves $L - 2d/v$ for queueing and compute. This is the concrete form of "the network makes the use case impossible": not slowness in general, but a budget smaller than the round trip to the nearest place the computation could run.

### When the edge wins on latency

Let the device compute in $t_{\text{edge}}$ and the data centre in $t_{\text{cloud}}$, with $t_{\text{edge}} \ge t_{\text{cloud}}$ in the usual case of a weaker processor. The cloud path also has to move the request payload of $s$ bits over an uplink of bandwidth $B$. Then

$$T_{\text{edge}} = t_{\text{edge}} + q_{\text{edge}}, \qquad T_{\text{cloud}} = \frac{2d}{v} + \frac{s}{B} + q_{\text{cloud}} + t_{\text{cloud}},$$

and computing at the edge is faster exactly when

$$t_{\text{edge}} - t_{\text{cloud}} \;<\; \frac{2d}{v} + \frac{s}{B} + \left(q_{\text{cloud}} - q_{\text{edge}}\right).$$

In words: the edge wins when the extra time its slower processor costs is less than the network time it saves. The left side is the compute penalty and shrinks as the work is made smaller or the device faster; the right side is the network saving and grows with distance, with payload size and with congestion. Setting $t_{\text{edge}} = t_{\text{cloud}}$ shows the edge wins outright whenever the device can do the work as fast as the server, since the right side is never negative once queueing is equal.

## Where it is used

[[Cloud Computing]] and [[Self-Hosting]] answer the other hosting question, who owns and runs the machines, and each combines with either end of this note's axis, so a placement decision is only fully stated when both answers are given. [[Model Inference]] is the machine-learning case: what a model asks of a device when inference moves onto it, and why the model usually has to be shrunk to fit first, which is the job of [[Model Compression]]. Any system that splits work between a device and a data centre is a [[Distributed System]], since the two share no memory and talk only by message; putting servers in several regions so packets need not cross the world, one of the reasons that note gives for distributing at all, is the same move as this one stopped short of the device. [[Request-Driven Communication]] supplies the availability product the offline argument above rests on. [[Scalability]] changes shape for work done on the device, because each new user arrives with their own processor, so that work stops being part of the load the operator's fleet has to absorb.
