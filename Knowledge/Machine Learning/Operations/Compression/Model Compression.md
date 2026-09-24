---
note_kind: concept
aliases:
  - model compression
  - compressed model
  - compressed models
  - network compression
  - neural network compression
  - model size reduction
up: "[[Model Inference]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## Definition

Model compression is making a fitted model smaller, fewer parameters or fewer bits per parameter, so that storing it, shipping it and running it cost less. It changes the artifact, not the task: the compressed model answers the same question as the original, and whatever accuracy it loses on the way is the price.

It is one of three answers to a model that takes too long to generate predictions, and the three attack different things:

- **Make inference itself faster** without touching what the model computes. This is inference optimization, and it works on how the arithmetic is scheduled rather than on how much of it there is.
- **Make the model smaller.** This is compression, and it works on how much arithmetic and memory the model needs in the first place. Four techniques do it, each attacking a different factor of the size, and they are listed with that factor under the formal statement.
- **Make the hardware it runs on faster**, leaving the model and its schedule alone and buying more arithmetic per second.

The three compose rather than compete. A quantized model compiled for an accelerator that does narrow integer arithmetic natively is all three at once, and the gains multiply only because each one attacks a factor the other two leave alone.

### Making inference faster without shrinking the model

A trained model is a computational graph of operators, and a framework normally runs each operator as a separate kernel that reads its inputs from memory and writes its output back. A deep learning compiler takes that graph, lowers it to an intermediate representation, and rewrites it into a functionally equivalent program that runs faster on one specific target. TVM (Chen and colleagues, OSDI 2018) is the reference design and does the work at two levels. At the graph level the main rewrite is **operator fusion**: several operators are merged into one kernel so the intermediate results never make the round trip to memory, which the authors measure at 1.2 to 2 times faster for the fused operators. At the operator level the question is the *schedule*, the loop order, tiling, threading and memory layout of one operator on one piece of hardware, and the number of valid schedules is too large to search by hand. The answer in AutoTVM ("Learning to Optimize Tensor Programs", Chen and colleagues, NeurIPS 2018) is to use machine learning on the compiler itself: a statistical cost model, gradient tree boosting by default, is trained on measured running times of candidate schedules and predicts which untried variants are worth measuring next, across a search space of billions of program variants. End to end, TVM reports speedups of 1.2 to 3.8 times over existing frameworks backed by hand-optimized libraries. Every one of those figures is a speedup in the sense $s = t_{\text{before}}/t_{\text{after}}$ for the same model with the same weights, which is what separates this route from compression: nothing about what the model computes has changed, so no accuracy is at stake.

### Faster hardware

The third route is an accelerator, a chip that gives up generality to do the dense multiply-adds of a neural network faster and more cheaply than a general CPU. The first-generation Tensor Processing Unit (Jouppi and colleagues, ISCA 2017) is the documented case: an inference chip built around a matrix unit of 65,536 multiply-accumulate cells working on 8-bit integers, with a peak of 92 tera-operations per second, measured on production inference workloads at 15 to 30 times faster than a contemporary Haswell CPU and K80 GPU and 30 to 80 times better in operations per watt. The detail worth carrying is the 8-bit: the hardware route and the compression route meet there, since a chip whose fast path is narrow integer arithmetic only pays off for a model whose weights have already been quantized to fit it.

## Formal statement

### The size a model occupies

The weights of a model with $P$ parameters stored at $b$ bytes each occupy

$$S = P \cdot b$$

bytes. That is the floor [[Scalability]] derives as $P \cdot b \cdot k$, evaluated at $k = 1$, since serving holds one copy of the weights and none of the gradient or optimizer state a fit carries; [[Distributed Training]] restates the same figure with $\beta$ for the bytes per parameter. At $P = 10^{8}$ and $b = 4$ (single precision) it is 0.4 GB, before any activation memory for the request being served.

The **compression ratio** is original size over compressed size,

$$C = \frac{S_{\text{orig}}}{S_{\text{comp}}} = \frac{P \cdot b}{P' \cdot b'}$$

so $C > 1$ means the model got smaller. The two factors in the fraction are independent, which is the whole taxonomy in one line: a technique either lowers the parameter count, lowers the bytes per parameter, or changes how the surviving parameters are stored. When two techniques attack different factors their ratios multiply, $C = (P/P') \cdot (b/b')$.

### Which factor each technique attacks

| technique | factor attacked | compressed size |
|---|---|---|
| [[Low-Rank Factorization]] | $P$: a $d_{\text{out}} \times d_{\text{in}}$ weight matrix becomes two factors of rank $r$ | $r(d_{\text{out}} + d_{\text{in}}) \cdot b$ per layer, smaller only when $r < d_{\text{out}}d_{\text{in}}/(d_{\text{out}} + d_{\text{in}})$ |
| [[Knowledge Distillation]] | $P$: a new, smaller student is trained to reproduce the teacher | $P_{\text{student}} \cdot b$, with the student's architecture free to differ entirely |
| [[Model Pruning]], structured | $P$: whole neurons, channels or filters removed, leaving a smaller dense model | $P' \cdot b$ |
| [[Model Pruning]], unstructured | stored nonzeros: individual weights zeroed, shape unchanged | $z \cdot b$ for the $z$ surviving values, plus the index overhead below |
| [[Model Quantization]] | $b$: each parameter stored in fewer bits | $P \cdot b'$, so float32 to int8 gives $C = 4/1 = 4$ on the weights |

Unstructured pruning is the row that needs care, because zeroing a weight does not remove it from a dense array. The saving exists only once the matrix is stored sparsely, and a sparse format pays for the position of every surviving value. In compressed sparse row form a $d_{\text{out}} \times d_{\text{in}}$ matrix with $z$ nonzeros and $b_{\text{idx}}$ bytes per index costs

$$S_{\text{CSR}} = z\,(b + b_{\text{idx}}) + (d_{\text{out}} + 1)\,b_{\text{idx}}$$

against $d_{\text{out}}d_{\text{in}}\,b$ dense. Ignoring the row pointers, the sparse form is smaller only when the density $z/(d_{\text{out}}d_{\text{in}})$ is below $b/(b + b_{\text{idx}})$, which is one half for 4-byte values with 4-byte indices. Size is also not speed here: the MobileNets authors note that unstructured sparse matrix operations are not typically faster than dense ones until a very high level of sparsity, so an unstructured-pruned model can be smaller on disk and no faster to run.

The measured case of ratios composing is AlexNet against SqueezeNet (Iandola and colleagues, 2016), in bytes needed to store all parameters: 240 MB for 32-bit AlexNet, 48 MB after SVD-based low-rank factorization ($C = 5$), 4.8 MB for SqueezeNet, an architecture designed small ($C = 50$), and 0.47 MB for SqueezeNet after pruning and 6-bit quantization on top ($C = 510$), with ImageNet top-1 accuracy of 57.2, 56.0, 57.5 and 57.5 percent respectively.

### Size is not latency

Parameter count and arithmetic are different quantities, and a size ratio is not a speed ratio. A fully connected layer does $d_{\text{out}}d_{\text{in}}$ multiply-adds per instance for $d_{\text{out}}d_{\text{in}}$ parameters, so there the two move together. A convolutional layer with a $D_K \times D_K$ kernel, $M$ input and $N$ output channels, applied over a $D_F \times D_F$ feature map, holds $D_K^{2}MN$ parameters and does

$$D_K^{2} \cdot M \cdot N \cdot D_F^{2}$$

multiply-adds, because the same small kernel is reused at every position. That is why, in the convolutional networks of the mid 2010s, most of the parameters sat in the fully connected layers while most of the running time sat in the early convolutional ones (Denton and colleagues 2014 report both), and why compressing the part that holds the parameters can leave the latency nearly where it was.

### The trade

Compression buys size and latency with accuracy, and the accuracy is only known by measuring it. The comparison is the same [[Performance Measure]] computed for the original $f$ and the compressed $f'$ on the same held-out [[Testing Set]], and the acceptance rule is a constrained choice: among the candidates that meet the size or latency budget, keep the smallest (or fastest) one whose drop

$$\Delta = \text{score}(f) - \text{score}(f') \le \varepsilon_{\text{acc}}$$

with $\varepsilon_{\text{acc}}$ set in advance. That is the satisficing-and-optimizing construction [[Production Machine Learning]] states for serving cost against accuracy in general, applied to one axis of it, and the threshold is chosen, not derived. An aggregate score understates what is lost. Hooker and colleagues ("What Do Compressed Deep Neural Networks Forget?", 2019) found pruned image classifiers matching the original's top-line accuracy while diverging sharply on a narrow subset of atypical and long-tail examples, so $\Delta$ computed over the whole test set can be near zero while it is large on a slice.

## Where it is used

[[Model Inference]] is the cost compression exists to reduce, the time and memory of turning one input into one prediction, and [[Batch and Online Prediction]] decides how much that cost matters: a caller blocked on an online prediction feels every millisecond, while a batch job mostly feels the bill. [[Edge Computing]] is where a model most often has to fit, a phone or an embedded board with a fixed memory and power budget, which turns the size $S$ from a cost into a hard constraint. [[Scalability]] owns the $P \cdot b \cdot k$ floor this note evaluates at $k = 1$, and [[Production Machine Learning]] is where accuracy against serving cost is already stated as a trade with no single optimum.

The four techniques, each a note of its own:

- [[Low-Rank Factorization]] replaces a large weight matrix or tensor with a product of smaller ones, cutting $P$ inside a layer.
- [[Knowledge Distillation]] trains a small student model to reproduce a large teacher's outputs, cutting $P$ by replacing the model outright.
- [[Model Pruning]] removes weights, or whole structures, that contribute little, cutting $P$ when structured and the stored nonzeros when not.
- [[Model Quantization]] stores and computes each parameter in fewer bits, cutting $b$.

[[Slice-Based Evaluation]] is the check the trade above needs, because the damage compression does concentrates on subsets an overall score averages away.

### Two taxonomies of the same field

[[DMLS Ch07 Model Deployment and Prediction Service|DMLS chapter 7]] lists four techniques: low-rank factorization, knowledge distillation, pruning and quantization. The survey by Cheng, Wang, Zhou and Zhang ("A Survey of Model Compression and Acceleration for Deep Neural Networks", 2017, later in *IEEE Signal Processing Magazine*) also has four categories, but cut differently: parameter pruning and quantization, low-rank factorization, transferred/compact convolutional filters, and knowledge distillation. Two differences follow. The survey keeps pruning and quantization together as one category, both being ways of removing redundancy from existing parameters, where the DMLS list separates them, and the table above supports the separation, since they attack different factors of $S$. And the survey gives compact convolutional filters a category of its own, separate from low-rank factorization, on the grounds that they are a way of designing the architecture small and training it from scratch, applicable to convolutional layers only, whereas low-rank factorization can be applied to a pretrained model and to fully connected layers as well. The DMLS list files compact filters under low-rank factorization instead; why that is at best partly right is argued in [[Low-Rank Factorization]].
