---
note_kind: concept
aliases:
  - model inference
  - prediction latency
  - inference latency
  - inference time
  - prediction throughput
  - inference throughput
  - atomic prediction
  - bulk prediction
  - on-device inference
  - on-device machine learning
  - on-device ML
  - edge inference
  - ML on the edge
  - machine learning on the edge
  - ML in the browser
  - in-browser inference
up: "[[MLOps]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## Definition

Model inference is running a fitted model on an input to get its prediction back: the forward pass, with every weight already fixed, so nothing is learned by doing it. Statistical inference is a different word spelled the same, and it means very nearly the opposite: using data to estimate the parameters of the distribution that generated it, or to draw a conclusion about a population from a sample, which Wasserman states as the process of using data to infer the distribution that generated the data and equates outright with what computer science calls learning (*All of Statistics*, 2004, section 6.1). That is the fitting half, not the serving half. [[Missing Value Imputation]] and [[Feature Generalization]] use the word in the statistical sense, when they speak of a mechanism being ignorable "for inference", and neither reading may be substituted for the other.

## Formal statement

**The call.** With parameters $\boldsymbol\theta^{*}$ fixed by a fit that has already finished, inference is the map

$$\hat y = h_{\boldsymbol\theta^{*}}(\mathbf x)$$

evaluated on an input $\mathbf x$ that arrived after the fit. The call reads $\boldsymbol\theta^{*}$ and never writes it; a call that changed the parameters would be a training step, which is [[Online Learning]].

**Latency and throughput.** Both are measured on the fitted model, not on the fit, and they are two different quantities.

- *Prediction latency* is the elapsed time $L$ from handing the model an input to having its prediction. It varies from call to call, so it is a distribution rather than a number, and what gets reported is a percentile of it:

$$L_{(q)} = \inf\{\, t : \Pr[L \le t] \ge q \,\}$$

  ex $L_{(0.9)}$, the time nine calls in ten finish within. scikit-learn 1.6's user guide on computational performance defines latency the same way and names the 90th percentile as the kind of figure operations engineers watch.
- *Prediction throughput* is the number of predictions delivered per unit time, $X$ predictions per second.

The two are tied by how many inputs one call carries. An *atomic* prediction passes one input per call; a *bulk* prediction passes $b$ inputs in one call. A single caller issuing calls back to back gets

$$X_{1} = \frac{1}{L_{1}} \qquad\qquad X_{b} = \frac{b}{L_{b}}$$

where $L_b$ is the latency of one call carrying $b$ inputs. Bulk raises throughput exactly when $L_b < b\,L_1$, that is, when one call on $b$ rows costs less than $b$ calls on one row each. The same scikit-learn page reports that this holds for every estimator it measures, bulk mode being faster by up to one to two orders of magnitude, and attributes it to branch prediction, CPU caches and the linear algebra libraries doing better work on a whole matrix. What the bulk call gives up is that every one of the $b$ rows waits for the whole call: each row's latency is $L_b$, not $L_1$. That is the trade [[Production Machine Learning]] records serving stacks exposing as a knob.

**End-to-end latency.** The caller of a served model does not feel $L_{\text{inference}}$. It feels the sum of every stage between sending the request and reading the answer:

$$L = L_{\text{network}} + L_{\text{queue}} + L_{\text{features}} + L_{\text{inference}}$$

with $L_{\text{network}}$ the round trip to wherever the model runs (serialization of the request and response included), $L_{\text{queue}}$ the time the request waits for a free worker, and $L_{\text{features}}$ the time spent fetching or computing the inputs the request did not carry, which is where the batch and streaming features of [[Feature]] get read.

The network term has a physical floor. Light in optical fibre travels at about $v \approx 2 \times 10^{8}$ m/s, so a model a distance $d$ away costs at least

$$L_{\text{network}} \ge \frac{2d}{v} \approx 1 \text{ ms per } 100 \text{ km}$$

of round trip before any routing, queueing or processing; [[Edge Computing]] carries this bound. Whether the network or the model is the bigger bottleneck is then the inequality $L_{\text{network}} > L_{\text{inference}}$ and nothing else. A model that answers in 5 ms, served from a region 4,000 km of fibre from the caller, has a network floor of 40 ms, eight times its own latency, and no amount of work on the model moves the total much. A model that takes a second per call is bottlenecked by itself wherever it runs. So the claim that network latency is often the larger term is true exactly for fast models served from far away, and it is the reason moving the model onto the device removes a term rather than shrinking one.

**What the caller feels is a tail, not a mean.** When one request fans out to $n$ servers in parallel and waits for all of them, and each is slow with probability $p$ independently,

$$\Pr[\text{request slow}] = 1 - (1 - p)^{n}$$

Dean and Barroso ("The Tail at Scale", *CACM* 56(2), 2013) work it at $p = 0.01$ and $n = 100$, where 63 percent of requests are slow although each server is slow once in a hundred calls. [[Data Parallelism]] carries the same arithmetic for a synchronous training step. This is why a latency target is stated as $L_{(q)} \le$ budget at a high $q$, never as a bound on the mean.

**Three levers on $L_{\text{inference}}$.** Make the model smaller, which is [[Model Compression]]; make the same model run faster, by compiling and optimizing its computation for the hardware it lands on; or run it on faster hardware. The first changes the model and can change its predictions, the other two change only the time.

### Where inference runs

Inference either runs in the cloud, on machines in a public or private datacenter reached over a network ([[Cloud Computing]]), or on the device the user holds, a phone, a laptop, a browser tab, a watch, a car, a security camera, a robot or an embedded board, which is [[Edge Computing]] with a model in it. The generic case for the edge, working offline and keeping data off the network, is carried there. What is specific to a model is the latency term above: on the device $L_{\text{network}} = 0$, so the whole budget goes to $L_{\text{inference}}$.

A list of edge devices sometimes puts FPGAs and ASICs next to phones and cars. They are kinds of chip, not devices anyone owns, and either can sit inside a consumer device or inside a datacenter server, so they say what the computation runs on and not where; [[Edge Computing]] sets this out.

What a device asks of a model is what the datacenter never did:

- **Memory.** A model with $P$ parameters stored at $B$ bits each occupies

$$\text{size} = \frac{P \cdot B}{8} \text{ bytes}$$

  so 100 million parameters take 400 MB in 32-bit floats and 100 MB in 8-bit integers. [[Model Quantization]] is the lever that moves $B$.
- **Integer-only hardware.** Some edge accelerators compute only in fixed-point arithmetic, so a model that needs floating point cannot run on them at all until it is quantized.
- **Battery.** Every operation costs energy the user pays for, so a model that fits and runs can still be too expensive to run often.

Together these are why [[Model Compression]] exists as a family rather than as an optimization nobody needs.

The runtimes that load a model onto a device have moved since 2022, and a pre-2024 reference will not match current names (all checked September 2026):

| runtime | owner and target | status |
|---|---|---|
| LiteRT | Google; Android, iOS, embedded | the new name for TensorFlow Lite, announced on the Google Developers Blog on 4 September 2024 |
| ExecuTorch | PyTorch; mobile, embedded, desktop | replaces PyTorch Mobile, whose pytorch.org page now redirects to the ExecuTorch documentation; version 1.0 announced on the PyTorch blog on 24 October 2025 |
| TensorRT | NVIDIA; NVIDIA GPUs only | an inference optimizer and runtime for NVIDIA hardware, so it runs on a device only when that device carries an NVIDIA GPU, which in practice means a Jetson-class embedded board |
| Core ML | Apple; Apple devices | Apple's framework for running a model inside an app, with `coremltools` converting from PyTorch, TensorFlow and scikit-learn, among others |
| ONNX Runtime | Microsoft, open source; servers, mobile, browser | runs any model exported to the ONNX format, which is the route out of scikit-learn below |

### In the browser

A browser is the one edge target that needs no install, and the price is that the model runs inside the browser's sandbox rather than natively. There are two routes in. One runs the computation as JavaScript, which is how TensorFlow.js began, with WebGL for the GPU. The other compiles it to WebAssembly, a low-level portable binary format designed as a compilation target for languages like C, C++ and Rust, validated and executed in the same sandbox as JavaScript (Haas et al., "Bringing the Web up to Speed with WebAssembly", PLDI 2017).

How slow is slow is measured, not guessed. Write the slowdown of a program as $s = T_{\text{Wasm}} / T_{\text{native}}$. Haas et al. reported, as Jangda and colleagues summarize them, seven of 24 small scientific kernels within 10 percent of native ($s \le 1.1$) and almost all under $s = 2$. Jangda, Powers, Berger and Guha ("Not So Fast: Analyzing the Performance of WebAssembly vs. Native Code", USENIX ATC 2019) ran the SPEC CPU suite instead and found a mean $s$ of 1.45 in Firefox and 1.55 in Chrome, with peaks of 2.08 and 2.5: real programs pay about half again their native time, more than the kernels suggested. These are 2019 browsers and the figures date with them.

"Much faster than JavaScript" holds against the right baseline and depends on which one. Against asm.js, the typed subset of JavaScript that compilers emitted before WebAssembly existed, Jangda et al. measure WebAssembly faster by $1.54\times$ in Chrome and $1.39\times$ in Firefox, which is faster but not "much". The large ratios come from comparing different code: TensorFlow.js documents its WebAssembly backend as 10 to 30 times faster than its plain JavaScript CPU backend, but the WebAssembly backend runs the XNNPACK kernel library while the JavaScript one does not, so the ratio measures an optimized library against an unoptimized one as much as it measures the format.

Two browser APIs postdate the 2022 picture and change what "slow in browsers" means, because both reach hardware WebAssembly cannot. **WebGPU** gives a page the GPU for general computation: it shipped in Chrome 113 in 2023, in Firefox 141 on Windows in July 2025 and in Safari 26 in September 2025, and web.dev declared it supported in all major browsers on 25 November 2025. **WebNN**, the Web Neural Network API, lets a page hand a whole network to the operating system's CPU, GPU or NPU path; it is a W3C Candidate Recommendation Draft dated 10 September 2026, in a Chrome origin trial for milestones 147 to 149, and not shipped by default in any browser as of September 2026. ONNX Runtime Web already exposes all of these as backends, WebAssembly, WebGL, WebGPU and WebNN, with WebAssembly the only one that supports every ONNX operator.

### In scikit-learn

In scikit-learn 1.6, `predict`, `predict_proba`, `decision_function` and `transform` on a fitted estimator are inference; `fit` and `partial_fit` are not. Passing a matrix of $b$ rows is bulk prediction and passing one row at a time is atomic, and the relation $X_b = b / L_b$ above is why a batch job should call `predict` once on everything rather than in a loop.

Every call also validates its input, and the check that every value is finite is a full pass over the data. When the caller already guarantees finite inputs, `sklearn.set_config(assume_finite=True)`, or the environment variable `SKLEARN_ASSUME_FINITE` set before import, skips it; `sklearn.config_context(assume_finite=True)` does the same inside a `with` block.

A fitted estimator reaches a server by being persisted, and the 1.6 model persistence page lists five formats with their risks:

- `pickle`, `joblib` and `cloudpickle` serialize the Python object itself, need the same environment on both sides, and execute arbitrary code when loaded, so the page says never to load one from an untrusted source.
- `skops.io` is more secure than the pickle family and can be partly inspected without loading, at the price of speed and type coverage.
- ONNX, via `skl2onnx`, drops the Python object entirely, so the serving side needs no Python and no scikit-learn, at the price of supporting only some estimators; the page calls it the most secure option and still recommends serving it in a sandbox.

**scikit-learn has no serving layer.** It has no endpoint, no request batching, no queue and no model server: the persistence page ends at loading the artifact and says only that serving it may mean deploying it as a web service in a container. Everything between the request and `predict` is built outside the library.

## Where it is used

[[Production Machine Learning]] is the setting that makes this a concern at all, and its computational-priority row names single-prediction latency as what production optimizes; the quantities it uses in passing are defined here. [[Batch and Online Prediction]] is the decision of when inference runs, ahead of any request or when the request arrives, and online prediction is the mode in which the latency sum above is a user-facing budget. [[Model Compression]] is the first of the three levers on $L_{\text{inference}}$, and [[Model Quantization]] its member that also decides whether a model fits on integer-only hardware at all.

[[Edge Computing]] is where the network term drops to zero, and it carries the physical floor this note uses. [[Cloud Computing]] is the other place inference runs, where the model is reached over the network and that floor is paid on every call. [[MLOps]] is the practice that owns a fitted model from this point on, and inference is its most frequent single operation. [[Data Parallelism]] carries the same tail-latency arithmetic on the training side.
