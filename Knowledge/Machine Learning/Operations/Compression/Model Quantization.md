---
note_kind: method
aliases:
  - model quantization
  - weight quantization
  - post-training quantization
  - PTQ
  - quantization-aware training
  - QAT
  - low-precision inference
  - fixed-point inference
  - integer quantization
  - INT8 quantization
  - low-precision training
  - mixed-precision training
  - mixed precision training
  - reduced precision
  - quantized model
up: "[[Model Compression]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## What it does and when

Model quantization stores a fitted model's weights, and usually its activations too, in fewer bits than the 32 of single-precision floating point, most often as 8-bit integers, and where the hardware allows it also does the arithmetic in that narrower format. It is not binning. [[Feature Distribution Transformation]] also rounds values onto a grid, but it rounds a feature's values before anything is fitted and the model learns from the bucket indices; quantization rounds the parameters of a model that has already been fitted, and the question is how little the model's output moves. Rounding onto a grid is the only thing the two share. "Precision" throughout this note means numeric precision, the bit width of a number, and has nothing to do with [[Precision]] the classification metric.

What it buys is three separate things, and only the first is unconditional. The weights get smaller by a factor of $32/b$ for $b$ bits, which is a change to the bytes per parameter in the $P \cdot b \cdot k$ floor of [[Scalability]] and nothing else. Less memory per stored number leaves room for a larger batch when training. And the model can run faster, but only on hardware that has fast arithmetic at the narrow width; the size saving is a fact about storage and the speed saving is a fact about the chip, which is the distinction the formula section makes exact.

Reach for it first among the [[Model Compression]] techniques when a model is too large or too slow to serve, because it asks for the least: no change to the architecture, which [[Low-Rank Factorization]] needs, no teacher, which [[Knowledge Distillation]] needs, and no decision about which weights to throw away, which [[Model Pruning]] needs. Every tensor in every architecture is a tensor of numbers that can be rounded. That is the sense in which it is the most general of the four, and the claim holds best at 8 bits: the survey by Gholami, Kim, Dong, Yao, Mahoney and Keutzer ("A Survey of Quantization Methods for Efficient Neural Network Inference", 2021) calls it straightforward with current methods to quantize and deploy models to INT8 without losing accuracy, says quantization and pruning keep performance at 4 times compression or more where distillation tends to lose accuracy under aggressive compression, and puts the memory and latency reductions realized in practice at 4 to 8 times. Two limits come with that. Below 8 bits the generality thins out: the same survey says software for lower bit widths is not widely available and sometimes does not exist. And "most commonly used" is a ranking the survey does not measure, so it is best held as a judgement about how routine INT8 has become rather than a counted fact.

### Three regimes, not two

Quantization "during training" names two different things, and quantization "after training" a third. They differ in what gets rounded, when, and what is kept at full precision, and they buy different things.

| regime | what happens | what is kept at full precision | what it buys |
|---|---|---|---|
| **post-training quantization** (PTQ) | a model fitted in single precision is quantized afterwards, with a small calibration set run through it to measure the range of each activation | nothing, at inference | a smaller, possibly faster model for serving, at almost no cost |
| **quantization-aware training** (QAT) | during training, or fine-tuning, the forward pass rounds weights and activations exactly as the inference engine will, so the weights learn to sit where rounding hurts least (Jacob and colleagues, CVPR 2018) | the weights and all arithmetic, during training | the same serving model as PTQ with less accuracy lost, paid for with a training run |
| **low-precision and mixed-precision training** | the training arithmetic itself runs in 16 or 8 bits to save memory and time, while a single-precision master copy of the weights absorbs the updates (Micikevicius and colleagues, "Mixed Precision Training", ICLR 2018) | the master weights, and the accumulations | a cheaper training run; the model it produces is not thereby quantized for serving |

QAT and mixed-precision training both happen "during training" and have opposite aims. QAT trains in floating point while pretending to be quantized, so that the product can be quantized later; mixed-precision training really is computing in fewer bits, to make the training itself cheaper. BinaryConnect (Courbariaux, Bengio and David, NeurIPS 2015) is the far end of the second idea: weights constrained to $\pm 1$ in the forward and backward passes, with real-valued stored weights accumulating the gradients. Every one of them keeps a full-precision copy somewhere, because a gradient step is usually far smaller than the gap between two representable values, and a weight stored only at low precision would round each update away.

That master copy is why the claim "fewer bits per parameter, so a larger model fits on the same hardware" is true of training only in a qualified form. By the accounting [[Scalability]] and [[Distributed Training]] already carry from ZeRO (Rajbhandari, Rasley, Ruwase and He, SC 2020), single-precision Adam holds four 4-byte copies of every parameter and mixed-precision Adam holds a 2-byte weight, a 2-byte gradient and 12 bytes of single-precision master weights and moments:

$$4 \cdot 4 = 16 \text{ bytes} \qquad \text{against} \qquad 2 + 2 + 12 = 16 \text{ bytes}$$

The per-parameter state does not shrink at all, and the weights alone grow from 4 bytes to $2 + 4 = 6$, which Micikevicius and colleagues state as 50 percent more weight memory than single-precision training. What shrinks is the activation memory, stored in half precision, which dominates a training run at realistic batch sizes, and they report overall training memory roughly halved. So the room for a larger model or a larger batch comes through the activations, not through a cheaper parameter. The per-parameter saving is real only where no master copy is kept, which is inference, where $k = 1$ and the quantized weights are all there is.

### Where it is the standard, and what hardware does now

Integer-only inference is the arrangement in which every multiply and add at serving time is done on integers, with no floating point anywhere in the inference path. Jacob, Kligys, Chen, Zhu, Tang, Howard, Adam and Kalenichenko ("Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference", CVPR 2018) is the design the mobile runtimes adopted, and the formula section shows why it is possible. Some hardware accepts nothing else, which makes quantization a requirement rather than an optimization there. Google's Coral documentation (checked 2026-09-24) requires Edge TPU models to have tensor parameters quantized to 8-bit fixed-point numbers, int8 or uint8, produced by either quantization-aware training, which it recommends, or full integer post-training quantization. The Gholami survey adds microcontrollers built on ARM Cortex-M cores without a floating-point unit, and the GAP-8 RISC-V edge chip, as processors that support only integer arithmetic. That is the [[Edge Computing]] case, where fitting the model in memory and power is a hard constraint.

Hardware support for low-precision training has moved a long way since 2022. The formats, with their papers:

- **bfloat16**: 8 exponent bits like single precision and 7 stored mantissa bits, so the same range with less resolution. Kalamkar and colleagues ("A Study of BFLOAT16 for Deep Learning Training", 2019) train across vision, speech, language and recommendation with no hyperparameter changes, where IEEE half precision needs loss scaling.
- **FP8**: Micikevicius and colleagues ("FP8 Formats for Deep Learning", 2022) define two encodings, E4M3 (4 exponent, 3 mantissa bits, maximum 448) for weights and activations and E5M2 for gradients, and match 16-bit training results on models up to 175 billion parameters with the hyperparameters left unchanged.
- **4-bit floating point**: FP4 E2M1, and block-scaled variants.

And the hardware, each checked 2026-09-24 against the vendor's own documentation: NVIDIA's Transformer Engine supports FP8 training and inference on Hopper, Ada and Blackwell GPUs, and MXFP8 and NVFP4 on Blackwell only. AMD's ROCm documentation lists FP8 on the Instinct MI300 series (in the FNUZ variant) and MX-format FP8, FP6 and FP4 on the MI350 series. The Gholami survey's 2021 verdict, that training much below half precision needs significant tuning, has therefore been overtaken for FP8 by hardware and recipes built for it; for 4-bit training the vendor documentation establishes support in the silicon, not that it is routine.

## Algorithm or formula

**Notation.** Here $b$ is the bit width, the convention of the quantization literature, so one number occupies $b/8$ bytes. [[Scalability]] and [[Model Compression]] write the bytes per parameter as $b$ and [[Distributed Training]] writes it as $\beta$; in this note the bytes are always $b/8$, and $\beta$ is the upper end of the clipping range. Scalars $x, q, s, z$; a weight vector $\mathbf{w}$; a weight matrix $\mathbf{W}$.

### The affine map

Choose a clipping range $[\alpha, \beta]$ that contains $0$. Divide it into $2^{b} - 1$ equal steps of width

$$s = \frac{\beta - \alpha}{2^{b} - 1}$$

the **scale**, and let the **zero point** $z = \operatorname{round}(-\alpha / s)$ be the integer that stands for the real number $0$. A real value $x$ becomes the $b$-bit unsigned integer

$$q = \operatorname{clamp}\!\big(\operatorname{round}(x / s) + z,\ 0,\ 2^{b} - 1\big)$$

and is read back, **dequantized**, as

$$\hat{x} = s\,(q - z)$$

Jacob and colleagues write the scheme as $r = S(q - Z)$, the same map; the Gholami survey writes the quantizer as $Q(r) = \operatorname{Int}(r/S) - Z$, with the sign of the zero point flipped, which is a convention and not a different method. Two reasons for insisting that $0$ is exactly representable, which is what the integer $z$ guarantees: zero padding in convolutions must stay exactly zero, and a ReLU output that is exactly zero should not come back as a small nonzero number.

The two things that can go wrong with a number are now exact.

- **Inside the range, the error is at most half a step.** For $\alpha \le x \le \beta$, rounding to the nearest grid point gives
  $$|x - \hat{x}| \le \frac{s}{2} = \frac{\beta - \alpha}{2\,(2^{b} - 1)}$$
  so each extra bit roughly halves the worst-case error. Under the usual model of rounding error as uniform over one step, the mean squared error is $s^{2}/12$.
- **Outside the range, the error is not bounded.** The clamp sends every $x > \beta$ to $q = 2^{b} - 1$, so $\hat{x} = \beta$ and $|x - \hat{x}| = x - \beta$, which grows without limit; likewise below $\alpha$. This is the precise form of the downside that fewer bits represent a smaller range: the range is chosen, not given, and whatever falls outside it is clipped rather than rounded.

The two error terms pull $[\alpha, \beta]$ in opposite directions. Widening the range to cover more values shrinks the clipping error and raises $s$, and with it the rounding error on everything inside. Choosing the range is therefore the substantive decision in quantization, and it has its own name, **calibration**. (This is not [[Model Calibration]], which is about whether predicted probabilities match observed frequencies.) The standard choices, from the survey: the observed minimum and maximum; a percentile of the observed values instead of the extremes, so that a few outliers are clipped rather than allowed to set $s$; the range minimizing the KL divergence between the real and quantized distributions; or the range minimizing the mean squared error. Weights are fixed after training, so their range is read off the weights. Activations depend on the input, so their range is either estimated once from a calibration set and frozen, **static** quantization, or recomputed for every input at run time, **dynamic** quantization, which is more accurate and costs a min and max over every activation on every request.

### Symmetric quantization

The special case $\alpha = -\beta$ with $\beta = \max_i |x_i|$ puts the zero point at $0$, and with signed integers the map becomes

$$q = \operatorname{clamp}\!\big(\operatorname{round}(x / s),\ -(2^{b-1} - 1),\ 2^{b-1} - 1\big), \qquad s = \frac{\beta}{2^{b-1} - 1}$$

This is the "restricted range" form, $[-127, 127]$ at 8 bits; Jacob and colleagues keep weights off $-128$ for the same reason. Symmetric is the usual choice for weights, which tend to be centred on zero, because a zero point of $0$ removes terms from the integer arithmetic below. It wastes half the grid on a tensor that is all one sign, and a ReLU output is the standard case, which is why activations are usually quantized with the affine form.

### Granularity

One $(s, z)$ can be shared by a whole tensor, **per-tensor** quantization, or each output channel $c$ of a weight matrix $\mathbf{W}$ can get its own,

$$s_c = \frac{\beta_c - \alpha_c}{2^{b} - 1}, \qquad \alpha_c = \min_j W_{cj},\ \beta_c = \max_j W_{cj}$$

**per-channel** quantization. The reason is that the rounding error bound $s/2$ is set by the widest channel: under per-tensor quantization a channel whose weights span a hundredth of the tensor's range gets a hundredth of the grid, and its weights round to a handful of levels. The survey calls per-channel the current standard for convolutional kernels, at negligible overhead, while finer groupings inside a channel add real overhead. Jacob and colleagues used one set of parameters per array.

### Why integer-only inference is possible

With both operands quantized, a dot product between an activation vector and one row of weights expands as

$$\hat{\mathbf{x}}^{\top}\hat{\mathbf{w}} = s_x s_w \sum_{j} (q_{x,j} - z_x)(q_{w,j} - z_w)$$

Everything inside the sum is integer arithmetic, 8-bit products accumulated into a 32-bit integer, and the single real multiplier $s_x s_w / s_y$ that rescales the result into the output's own grid is itself applied as an integer multiply and a bit shift. That is the whole trick of Jacob and colleagues, and a zero point of $0$ on the weights, symmetric quantization, deletes the cross terms that expanding the product would otherwise create.

### Training the weights to tolerate rounding

Quantization-aware training inserts the quantize-then-dequantize step $x \mapsto \hat{x}$, **fake quantization**, into the forward pass, so the loss is computed on what the integer model will actually compute, while the weights themselves stay in floating point. The rounding function has zero gradient almost everywhere, so the backward pass uses the **straight-through estimator**, treating the step as the identity inside the range and blocking the gradient outside it:

$$\frac{\partial \hat{x}}{\partial x} \approx \begin{cases} 1 & \alpha \le x \le \beta \\ 0 & \text{otherwise} \end{cases}$$

Jacob and colleagues set weight ranges from the weights' own minimum and maximum, track activation ranges during training with an exponential moving average, and switch activation quantization off for the first stretch of training, 50 thousand to 2 million steps in their account, so that the ranges settle before they start clipping.

### Size, batch and speed

**Size.** Against single precision, the stored weights shrink by

$$\frac{32}{b}$$

4 at INT8 and 8 at INT4, plus a small overhead of one scale and one zero point per tensor or per channel. The memory being shrunk is the floor that [[Scalability]] derives as $M = P \cdot (\text{bytes per parameter}) \cdot k$; quantization leaves $P$ and $k$ alone and changes only the bytes, from $4$ to $b/8$.

**Batch.** During training, the activations held for the backward pass cost about $n_B \cdot a \cdot b_{\text{act}}/8$ bytes for a batch of $n_B$ examples, each holding $a$ activation values at $b_{\text{act}}$ bits. On a device with memory $M_{\text{dev}}$, of which $M_{\text{params}}$ goes to the parameter state, the largest batch that fits is

$$n_B \le \frac{M_{\text{dev}} - M_{\text{params}}}{a \cdot b_{\text{act}}/8}$$

so halving $b_{\text{act}}$ from 32 to 16 roughly doubles the ceiling, which is where the larger batch comes from. $n_B$ is the per-worker batch size that [[Mini-Batch Gradient Descent]] writes as $b$.

**Speed.** In the roofline model (Williams, Waterman and Patterson, *Communications of the ACM*, 2009), a layer that does $F$ operations on $P$ weights takes at least

$$t \ge \max\!\left(\frac{P \cdot b/8}{B_{\text{mem}}},\ \frac{F}{R_b}\right)$$

on a device with memory bandwidth $B_{\text{mem}}$ and arithmetic rate $R_b$ at bit width $b$. Fewer bits always shrink the first term, since fewer bytes are moved. The second term shrinks only if $R_b$ is larger at the narrow width, which means the chip has integer or low-precision units and the runtime has kernels that use them. Where it does not, or where the model is run as **simulated quantization**, weights stored in low precision but dequantized to floating point before every operation, a compute-bound layer gains nothing, and the quantize and dequantize steps are extra work. Jacob and colleagues measured exactly that dependence: their integer MobileNets beat floating-point ones by about 10 percent accuracy at a fixed 33 ms budget on a Snapdragon 835 LITTLE core, and the gain was less noticeable on the Snapdragon 821, whose floating-point path is better optimized.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| bit width | $b$ | 8 (the Edge TPU requires it; ONNX Runtime's `quantize_static` defaults to `QInt8`) | halves the step $s$ and the error bound $s/2$ per extra bit, and multiplies the stored size by $b/32$ against float32. Going down from 8 is where accuracy starts to break and where kernels stop existing | start at 8, the width every runtime supports. Go below only with QAT and a target that has kernels at that width, and measure |
| granularity | $(s, z)$ per tensor or per channel $c$ | per tensor in ONNX Runtime (`per_channel=False`) | finer granularity gives each channel a grid fitted to its own range, so narrow channels keep their resolution; costs one $(s_c, z_c)$ per channel | per-channel for weights, which the survey calls the standard for convolution kernels; per-tensor for activations |
| symmetric or affine | $z = 0$ or free | ONNX Runtime: weights symmetric (`WeightSymmetric=True`), activations affine (`ActivationSymmetric=False`) | not a direction. Affine fits a lopsided range tightly; symmetric wastes up to half the grid on a one-signed tensor and simplifies the integer arithmetic | symmetric for weights, affine for activations after a ReLU |
| clipping range method | $[\alpha, \beta]$ | minimum and maximum (`CalibrationMethod.MinMax` in ONNX Runtime) | a tighter range (percentile, KL, MSE) lowers $s$ and the rounding error on typical values and raises the clipping error on the extremes | min and max for weights; for activations with heavy tails, compare percentile and entropy against min and max on a held-out set |
| calibration set | $n_{\text{cal}}$ examples | none; LiteRT recommends about 100 to 500 samples for full integer quantization (checked 2026-09-24) | a larger, more varied set estimates activation ranges closer to what production will send; past a few hundred the ranges stop moving | draw it from the same distribution as serving traffic, not from a convenient corner of the training set |
| static or dynamic activation ranges | | static in full integer quantization; dynamic in `quantize_dynamic` | not a direction. Dynamic is more accurate and adds a range computation on every request; static is free at run time and is the only option on integer-only hardware | static wherever the target requires full integer; dynamic for CPU serving where accuracy matters more than the overhead |
| PTQ or QAT | | PTQ | not a direction. QAT recovers accuracy that PTQ loses, most of all at low bit widths, and costs a training run with training data | PTQ first; QAT when the PTQ model fails its accuracy budget on the [[Testing Set]] and a slice check |

The EMA smoothing constant and the delay before activation quantization is switched on belong to QAT and are pinned from the recipe rather than tuned: close to 1 for the smoothing, and a delay long enough for ranges to settle, as Jacob and colleagues describe. The data type of the stored integer, `int8` against `uint8`, changes only where the zero point sits, not the grid, and is fixed by the target runtime.

## Failure modes

- **One outlier sets the scale for everything.** With min and max calibration, $s = (\beta - \alpha)/(2^{b} - 1)$ is set by the single largest value, so an activation that occasionally spikes, or one weight far from the rest, stretches the grid and crushes the ordinary values into a few levels. In the implementation below, a single weight of $100$ among ten thousand standard normal ones raises $s$ from $0.028$ to $0.41$ and leaves the other weights using 19 of the 256 levels instead of 222. Jacob and colleagues name outlier weights as one of the two reasons per-layer quantization loses accuracy. The fixes are a percentile or MSE range, per-channel parameters, or keeping the offending layer at higher precision.
- **The accuracy loss is concentrated on some inputs.** A quantized model can match the original's overall score and still disagree with it on a narrow set of inputs, typically the rare and atypical ones whose activations sit near the edge of the calibrated range. The overall $\Delta$ that [[Model Compression]] tests against is then near zero while it is large on a slice, which is exactly what [[Slice-Based Evaluation]] exists to catch; compare the quantized and original models slice by slice, and compare their predictions row by row, before shipping.
- **The quantized model is not faster.** Weights stored in 8 bits and dequantized to floating point before every multiply, or a device with no fast integer path, gives the size saving and none of the speed, and the ONNX Runtime documentation (checked 2026-09-24) says it is not rare to get worse performance on old hardware without the instructions for efficient int8 arithmetic, because quantizing and dequantizing has overhead. Measure latency on the target device, not on the development machine.
- **The calibration set does not look like production.** Static activation ranges are estimates from the calibration set and are frozen afterwards. Calibrate on daytime images and serve night ones, or calibrate on a clean benchmark and serve user uploads, and production activations fall outside $[\alpha, \beta]$, where the error is the unbounded clipping term. This is [[Nonrepresentative Training Data]] arriving through a hundred examples nobody thought of as training data.
- **Half-precision training without loss scaling.** Small gradients underflow to zero in IEEE half precision, which has a narrower range than single precision, and the model quietly stops learning; updating half-precision weights without a single-precision master copy loses updates the same way. Micikevicius and colleagues report an 80 percent relative accuracy loss on a speech model trained without the master copy. bfloat16 avoids the underflow by having single precision's range.
- **Going below 8 bits by PTQ alone.** Every error term scales with $s$, which doubles for each bit removed, and the survey reports PTQ losing more accuracy than QAT especially at low precision; below INT8 there may also be no kernel on the target to run the result.

## Implementation

**scikit-learn has no model quantization API**, and no reduced-precision inference or mixed-precision training either: its estimators fit and predict in float64 or float32 and nothing in the library rounds a fitted model's parameters. Quantization lives in the deep learning runtimes, named after the demonstration.

NumPy 2.x, run with NumPy 2.0.2 on Python 3.9: the affine map with its error bound asserted, per-tensor against per-channel on a matrix whose rows live on different scales, the unbounded error outside the range, and the outlier failure.

```python
import numpy as np


def affine_qparams(x, b, axis=None):
    """Scale s and zero point z for the affine map, over a min/max clipping range.

    axis=None gives one (s, z) for the whole tensor; axis=1 gives one per row,
    which is per-channel when rows are output channels.
    """
    alpha = np.minimum(x.min(axis=axis, keepdims=True), 0.0)  # the range must contain 0
    beta = np.maximum(x.max(axis=axis, keepdims=True), 0.0)
    s = (beta - alpha) / (2**b - 1)
    z = np.clip(np.round(-alpha / s), 0, 2**b - 1)
    return s, z, alpha, beta


def quantize(x, s, z, b):
    return np.clip(np.round(x / s) + z, 0, 2**b - 1).astype(np.int32)


def dequantize(q, s, z):
    return s * (q - z)


rng = np.random.default_rng(42)
b = 8

# a weight matrix W whose 64 output channels (rows) live on very different scales
row_scale = np.geomspace(0.01, 1.0, 64)[:, None]
W = (rng.standard_normal((64, 256)) * row_scale).astype(np.float32)

# per-tensor: one (s, z) for all of W
s, z, alpha, beta = affine_qparams(W, b)
W_hat = dequantize(quantize(W, s, z, b), s, z)
err = np.abs(W - W_hat)
assert np.all(err <= s / 2 + 1e-6), "rounding error inside the range exceeds s/2"
print(f"per-tensor : s = {s.item():.5f}, z = {int(z.item())}, max error = {err.max():.5f} <= s/2 = {s.item() / 2:.5f}")

# per-channel: one (s_c, z_c) per row
s_c, z_c, _, _ = affine_qparams(W, b, axis=1)
W_hat_c = dequantize(quantize(W, s_c, z_c, b), s_c, z_c)
err_c = np.abs(W - W_hat_c)
assert np.all(err_c <= s_c / 2 + 1e-6)
rel = lambda e: np.mean(e[:8]) / np.mean(np.abs(W[:8]))  # the eight smallest-scale rows
print(f"relative error on the smallest rows: per-tensor {rel(err):.3f}, per-channel {rel(err_c):.4f}")

# outside the clipping range the error is not bounded by s/2: it grows with the distance
x_out = np.array([beta.item() + 0.5, beta.item() + 5.0], dtype=np.float32)
x_out_hat = dequantize(quantize(x_out, s, z, b), s, z)
print("outside the range, error =", np.round(np.abs(x_out - x_out_hat).ravel(), 3))

# one outlier weight stretches s and crushes everything else onto a few levels
w = rng.standard_normal(10_000).astype(np.float32)
w_out = w.copy()
w_out[0] = 100.0
for name, v in [("no outlier", w), ("one outlier", w_out)]:
    s1, z1, _, _ = affine_qparams(v, b)
    levels = np.unique(quantize(v[1:], s1, z1, b)).size
    print(f"{name:11s}: s = {s1.item():.4f}, distinct levels used by the other 9,999 weights = {levels}")

# size against single precision: 32 / b
print("compression against float32:", W.nbytes / quantize(W, s, z, b).astype(np.uint8).nbytes)
```

Output:

```text
per-tensor : s = 0.02306, z = 116, max error = 0.01153 <= s/2 = 0.01153
relative error on the smallest rows: per-tensor 0.538, per-channel 0.0068
outside the range, error = [0.494 4.994]
no outlier : s = 0.0285, distinct levels used by the other 9,999 weights = 222
one outlier: s = 0.4067, distinct levels used by the other 9,999 weights = 19
compression against float32: 4.0
```

Per-tensor quantization meets its bound and still destroys the small rows, an average error of more than half their magnitude, because their bound is set by the largest row; per-channel brings that under one percent. The out-of-range error is $x - \beta$, less the fraction of a step by which rounding $z$ moved the grid's top end.

The real APIs, each checked 2026-09-24:

- **PyTorch.** `torch.ao.quantization` is being retired. The PyTorch 2.14 documentation says all quantization development is centralized in **torchao**, eager-mode users should move to `torchao.quantization.quantize_`, FX graph mode users to torchao's PT2E flow, and that PyTorch planned to delete `torch.ao.quantization` in 2.10 or at the earliest version once blockers were cleared; the 2.14 page still exists but documents only three error-measuring utilities. The current path is `quantize_(model, config)` with config objects such as `Int4WeightOnlyConfig` for weight-only schemes, and for export to a device the PT2E flow: `torch.export.export`, then `prepare_pt2e(model, quantizer)` from `torchao.quantization.pt2e.quantize_pt2e`, a loop over calibration data, and `convert_pt2e`. PT2E also has a QAT variant that inserts fake-quantize nodes.
- **ExecuTorch**, PyTorch's on-device runtime (stable docs 1.3), uses torchao's PT2E flow with a backend-specific quantizer, `XNNPACKQuantizer` for mobile CPUs.
- **LiteRT**, the runtime formerly called TensorFlow Lite (renamed on 2024-09-04). Post-training quantization is set on the converter: `converter.optimizations = [tf.lite.Optimize.DEFAULT]` alone gives dynamic-range quantization, 8-bit weights with no calibration data; adding `converter.representative_dataset` and `converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]` gives full integer quantization, the variant integer-only devices and the Edge TPU need; `converter.target_spec.supported_types = [tf.float16]` gives float16. Its own figures are 4 times smaller with 2 to 3 times speedup for dynamic range and 4 times smaller with 3 times or more for full integer, which are the vendor's figures for its own product.
- **TensorRT**, NVIDIA's inference SDK for its GPUs (latest 11.3.0). It supports INT8, INT4 weight-only, FP8 E4M3 and FP4 E2M1. Since 11.0 it takes only explicitly quantized networks, with quantize and dequantize nodes in the graph: the INT8 calibrator classes (`IInt8Calibrator` and its subclasses) and implicit quantization were deprecated in the 10.x line and removed in 11.0, and NVIDIA points to its TensorRT Model Optimizer for PTQ and QAT recipes.
- **ONNX Runtime** (`onnxruntime.quantization`, signatures read from the `main` branch): `quantize_dynamic(model_input, model_output, ..., per_channel=False, weight_type=QuantType.QInt8)` needs no data, and `quantize_static(model_input, model_output, calibration_data_reader, quant_format=QuantFormat.QDQ, per_channel=False, activation_type=QuantType.QInt8, weight_type=QuantType.QInt8, calibrate_method=CalibrationMethod.MinMax, ...)` takes a `CalibrationDataReader` and supports MinMax, Entropy and Percentile calibration. Its documented scheme is `val_fp32 = scale * (val_quantized - zero_point)`, the affine map above.

"Post-training quantization for free with a few lines of code" holds for LiteRT, whose converter flags are exactly that, and for ONNX Runtime's `quantize_dynamic`. It no longer describes PyTorch Mobile, which PyTorch's own pages mark as no longer actively supported in favour of ExecuTorch, or TensorRT, which since 11.0 expects a model already carrying quantize and dequantize nodes, produced by a separate tool. And TensorRT is on-device only in the sense that NVIDIA's Jetson boards (Orin and Thor in the current support matrix) run it; everywhere else it is a data-centre GPU runtime. Which runtime to serve with at all is the business of [[Model Inference]].
