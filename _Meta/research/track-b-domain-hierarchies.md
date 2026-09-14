---
note_kind: meta
title: "Track B: domain hierarchies in textbooks and graduate courses"
date: 2026-09-14
---

A reference to check a proposed tree against, not a proposed structure. About 3,800 words by word count, most of it in tables: eight textbooks and thirteen course offerings at chapter and lecture level. Abbreviations: PML1 (Murphy 2022), PML2 (Murphy 2023), ESL (Hastie et al 2e), GBC (Goodfellow et al 2016), PRML (Bishop 2006), B24 (Bishop and Bishop 2024), HOML (Géron 3e), MML (Deisenroth et al). Courses: CS229 (Stanford; 2018, Winter 2025, Summer 2026, plus the August 2026 main notes), CS189 (Berkeley Spring 2025), 10-701 (CMU Spring 2026, plus 2024, 2021, 2015 offerings), 10-601 (CMU Spring 2026), 10-715 (CMU Fall 2018), CS231n (Spring 2026), CS224n (Winter 2026), CS4780 (Cornell 2018), 6.390 (MIT Fall 2026).

## 1. Textbook top-level divisions

Confidence: high; B24 appendix titles are the one gap.

| Book | Parts | Ch | Appendices |
|---|---|---|---|
| PML1 | I Foundations (2 to 8), II Linear Models (9 to 12), III Deep Neural Networks (13 to 15), IV Nonparametric Models (16 to 18), V Beyond Supervised Learning (19 to 23) | 23 | A Notation |
| PML2 | I Fundamentals (2 to 6), II Inference (7 to 13), III Prediction (14 to 19), IV Generation (20 to 26), V Discovery (27 to 33), VI Action (34 to 36) | 36 | none |
| ESL | none | 18 | none |
| GBC | I Applied Math and ML Basics (2 to 5), II Modern Practices (6 to 12), III Research (13 to 20) | 20 | none |
| PRML | none | 14 | A Data Sets, B Distributions, C Matrices, D Calculus of Variations, E Lagrange Multipliers |
| B24 | none; "bite-sized chapters" in curriculum order: 1 Revolution, 2 Probabilities, 3 Distributions, 4 Single-layer Regression, 5 Single-layer Classification, 6 Deep Networks, 7 Gradient Descent, 8 Backpropagation, 9 Regularization, 10 CNNs, 11 Structured Distributions, 12 Transformers, 13 GNNs, 14 Sampling, 15 Discrete Latent Variables, 16 Continuous Latent Variables, 17 GANs, 18 Flows, 19 Autoencoders, 20 Diffusion | 20 | A, B, C exist (cited by the authors' solutions manual for matrix derivatives, functional derivatives, Lagrange multipliers); titles not read |
| HOML | I Fundamentals (1 to 9), II Neural Networks and Deep Learning (10 to 19) | 19 | A Project Checklist, B Autodiff, C Special Data Structures, D TensorFlow Graphs |
| MML | I Mathematical Foundations (2 to 7), II Central ML Problems (8 to 12) | 12 | |

## 2. Course lecture ordering

Confidence: high for tabulated courses.

| Course | Sequence |
|---|---|
| CS229 Summer 2026 | 1 linear regression; 2 GD, evaluation, bias-variance; 3 regularization, CV; 4 logistic; 5 GLMs, SGD; 6 trees, bagging, kNN; 7 to 8 NN; 9 k-means, GMM; 10 EM; 11 PCA, ICA; 12 to 14 RL; 15 RL and LLMs |
| CS229 main notes (Aug 2026) | I supervised (linear, logistic, GLMs, generative, kernels, SVM); II deep learning; III generalization and regularization (bias-variance, double descent, CV, Bayesian); IV unsupervised; V generative and foundation models (diffusion, LoRA, RAG, LLMs, RLVR); VI RL and control |
| CS229 Winter 2025 | 2 linear regression; 3 regularization; 4 logistic; 6 to 10 NN block (MLP, backprop, optimization, transformers and LMs, CNNs); 11 trees; 12 boosting; 15 PCA and autoencoders; 16 k-means, GMM; 17 to 18 RL; 19 fairness and algorithmic bias; 20 ethics |
| CS189 Spring 2025 | 1 to 4 perceptron, GD, SVM; 5 optimization; 6 to 9 decision theory, GDA, LDA, MLE, eigenvectors; 10 to 11 least squares, logistic, Newton, ROC; 12 to 13 bias-variance, ridge, lasso; 14 to 15 trees, forests; 16 to 19 NN, backprop, CNNs; 20 to 21 PCA, SVD, k-means; 22 to 23 dropout, batchnorm, AdamW; 24 to 25 AdaBoost, kNN |
| 10-701 Spring 2026 | 1 function approximation; 2 trees; 3 kNN, model selection; 4 linear regression; 5 MLE/MAP; 6 optimization; 7 logistic; 8 regularization and generalization; 9 NN; 10 to 11 CNNs, RNNs; 12 embeddings; 13 transformers; 14 societal impacts and algorithmic bias; 15 clustering; 16 dimensionality reduction; 17 to 18 RL; 19 pretraining, in-context learning; 20 to 21 learning theory; 22 to 23 bagging, boosting; 24 SVMs; 25 kernels, GPs; 26 generative models |
| CS231n Spring 2026 | 2 linear classifiers, kNN; 3 regularization, SGD, Adam; 4 MLP, backprop; 5 to 6 CNNs; 7 RNNs; 8 transformers; 9 detection, segmentation, visualization; 11 distributed training; 12 self-supervised; 13 VAE, GAN; 14 diffusion; 16 vision-language; 17 world models; 18 human-centered AI |
| CS224n Winter 2026 | 2 word vectors; 3 backprop; 4 RNN LMs; 5 transformers; 7 pretraining; 8 post-training (RLHF, SFT, DPO); 9 prompting, PEFT; 10 agents, RAG; 11 evaluation; 12 to 13 reasoning; 15 interpretability; 16 social impacts |

The coordinator re-fetched CS229 Winter 2025 and 10-701 Spring 2026 and confirmed the rows above.

## 3. Organizing axes and their costs

Confidence: high for the axis, medium for the costs (our reading, not the authors').

| Source | Axis | What fragments |
|---|---|---|
| PML1 | Model family, foundations front-loaded, Part V by task | PCA (20) far from ridge (11.3); regularization in 4.5, 11.3 to 11.4, 13.5, 18.1.3; clustering (21) far from GMM/EM (3.5, 8.7.2) |
| PML2 | Purpose (infer, predict, generate, discover, act) | Same model in several parts: GLMs (15), latent factor models (28), VAEs (21); EM in 6.5.3, 10.1.3, 10.3.5, 29.4.1 |
| ESL | Model family by increasing flexibility; one theory chapter (7) mid-book | Ensembles in 8.7, 8.8, 10, 15, 16; kernels in 5.8, 6, 12.3 |
| GBC | Pedagogical maturity within one family | PCA in 2.12, 13.1, 13.5; optimization in 4 and 8 |
| PRML | Bayesian paradigm, chapters by family | Regression (3) and classification (4) split; EM (9) apart from its users (12, 13) |
| B24 | Deep-learning curriculum, no parts | Classical ML compressed into 4 and 5; generative models spread over 17 to 20 with no header |
| HOML | Lifecycle (Part I), then tooling and architectures | Regularization in 4, 6, 11; gradient descent in 4, optimizers in 11; PCA in 8 and 17 |
| CS229, notes | Learning paradigm, family within | Regularization taught at L2 to 3 (2026) or L8 (2018) but filed in Part III after deep learning |
| CS189 | Discriminative versus generative, then difficulty | Regularization L13 and L22; eigen math L8 far from PCA L20; NN split L16 to 19 and L22 to 23 |
| 10-701 | Simple to complex, modern practice early | SVM and kernels L24 to 25 after RL; theory L20 to 21 after transformers; ensembles L22 to 23 far from trees L2 |
| CS231n | Pipeline, then architecture, then task | Regularization and optimization fused into L3 before the network exists (L4) |
| CS224n | LLM lifecycle | One ML-basics lecture; classical NLP absent |

**The central tension:** a tree picks one axis per level. Task-first breaks families. Family-first breaks tasks. Purpose-first (PML2) breaks both but absorbs new subfields best. Teaching order (every course) scatters each concept to the place it was first needed.

## 4. Cross-cutting concepts in textbooks (chapter.section)

Confidence: high, except B24 at chapter level only.

| Concept | PML1 | PML2 | ESL | GBC | PRML | B24 | HOML |
|---|---|---|---|---|---|---|---|
| Regularization | 4.5; 11.3, 11.4; 13.5; 18.1.3 | 26.6.5; 17 | 3.4, 5.8, 10.12, 18.3, 18.4 | 7; 14.2 | 3.1.4, 5.5 | 9 | 4, 6, 11 |
| Bias-variance | 4.7.6 | not in ToC | 2.9, 7.2, 7.3 | 5.4 | 3.2 | ch only | 6 |
| Optimization, GD | 8; 10.2.4, 10.3.3 | 6 | 11.4, 11.5 | 4.3, 8 | 5.2.4 | 7 | 4, 11 |
| Maximum likelihood | 4.2 | 24.2 | 8.2 | 5.5 | 2.3.4, 3.1.1, 4.2.2, 9.2.1 | | 4 |
| CV, model selection | 4.5.5, 5.2.2, 5.2.4, 5.4.3 | 3.8, 3.9 | 7 (7.10 CV) | 5.3, 11.4 | 1.3, 3.4 | | 1, 2, 3 |
| EM | 8.7.2, 20.2.3 | 6.5.3, 10.1.3, 10.3.5, 29.4.1 | 8.5 | 19.2 | 9, 12.2.2, 13.2.1 | 15, 16 | 9 |
| Kernels | 16.3, 17 | 18.2, 18.7 | 5.8, 6, 12.3 | none | 6, 7 | | 5 |
| Latent variables | 3.5, 20 | 28, 29 | 14 | 13, 14, 20 | 9, 12, 13 | 15, 16 | 9, 17 |
| Ensembles, boosting | 18 | 17.3.9 | 8.7, 8.8, 10, 15, 16 | 7.11 | 14 | none | 7 |
| Backpropagation | 13.3, 15.2.5 | 6.2, 9.7.1 | 11.4 | 6.5 | 5.3 | 8 | 10, App B |
| Graphical models | 3.6 | 4, 9 | 17 | 3.14, 16 | 8 | 11 | none |
| PCA, dim. reduction | 20 | 28.3, 32 | 14.5 to 14.9 | 2.12, 13, 14 | 12 | 16, 19 | 8, 17 |
| Evaluation metrics | 5.1.3 to 5.1.6 | 14.2, 20.4 | 7 | 11.1 | 1.5 | | 2, 3 |
| Linear model | 9 to 12 | 15 | 3, 4 | 5.7 | 3, 4 | 4, 5 | 4 |
| Attention, transformers | 15.4 to 15.6 | 16.2.7, 16.3.5, 22.4 | none | none | none | 12 | 16 |
| Bayesian inference | 4.6, 5.1, 10.5, 11.7, 13.5.5 | 3, 7 to 13, 17 | 7.7, 8.3, 8.6, 11.9 | 5.6, 17 to 19 | throughout | 14 | 9 |

## 5. Cross-cutting concepts in courses (lecture numbers)

| Concept | CS229 2026 / notes | CS189 | 10-701 |
|---|---|---|---|
| Regularization | 3 / ch 9 | 13, 22 | 8 |
| Bias-variance | 2 / 8.1 | 12 | 8 |
| Optimization, GD | 2, 5 / 1.1, 7.4 | 3, 5, 16, 23 | 6 |
| Maximum likelihood | 1 / 1.3, 4.1 | 7, 9, 12 | 5 |
| CV, model selection | 3 / 9.3 | 1, 13 | 3 |
| EM | 10 / ch 11 | none | none |
| Kernels | none / ch 5 | 4 | 25 |
| Ensembles | 6 / none | 15, 24 | 22, 23 |
| PCA | 11 / ch 12 | 8, 20, 21 | 16 |
| Linear model | 1, 4, 5 / ch 1 to 3 | 2, 10, 11 | 4, 7 |
| Transformers | none / 17.3 | none | 13 |
| Graphical models | none | none | none (present in 2015 and 2021 offerings, 10-715) |

## 6. Placement disagreements

Confidence: high.

| Topic | Placements |
|---|---|
| Logistic regression | Own chapter: PML1 10. Inside classification: ESL 4.4, PRML 4.3, B24 5. After regularized regression: HOML 4. Inside the regression lecture: CS189 L10. After MLE and optimization: 10-701 L7 |
| SVM | ESL 12 (late). PRML 7. PML1 17.3 (kernel methods, nonparametric part). HOML 5 (early). CS189 L3 to 4 (first classifier). 10-701 L24 (after RL). Dropped from CS229 2025 and 2026 lectures. Absent: GBC, B24 |
| PCA | Latent variables: PRML 12, B24 16. Dimensionality reduction: PML1 20, HOML 8. Unsupervised: ESL 14.5, CS189 L20. Linear algebra example: GBC 2.12, CS189 L8 and L21. With autoencoders: CS229 W25 L15 |
| Graphical models | A foundation: PML2 4. A family: PRML 8, B24 11. Advanced: GBC 16. Late specialty: ESL 17. A section: PML1 3.6. Absent: HOML and every 2025 to 2026 lecture schedule |
| Deep learning | A book: GBC, B24. A part: PML1 III (3 ch), HOML II (10 ch). A chapter: ESL 11, PRML 5. Interleaved by purpose: PML2. One block: CS229 W25 L6 to 10, 10-701 L9 to 13. Split: CS189 |
| Optimization | Foundations chapter: PML1 8, PML2 6, GBC 4 and 8, B24 7, 10-701 L6. Scattered: ESL 11.4, PRML 5.2, HOML 4 and 11, CS189 L3, 5, 16, 23 |
| Boosting | ESL 10 and 16.2 (additive models and regularization paths). PML1 18.5, PRML 14.3, HOML 7 (ensembles). GBC none |
| Learning theory | 10-715 L6 to 8 (early), 10-701 L20 to 21 (late), CS229 notes 8.3, absent from CS189 and every textbook except as ESL 7 |
| RL | A separate end unit everywhere (PML2 VI, HOML 18, CS229, 10-701 L17 to 18) yet RLHF, RLVR, PPO now tie it to LLMs (CS229 S26 L15, CS224n L8) |

## 7. Fairness, interpretability, causality, deployment

Confidence: high for titles; fairness in body text unchecked.

| Topic | Textbooks | Courses |
|---|---|---|
| Fairness, bias, ethics | **None in any ToC** (PML1, PML2, ESL, GBC, PRML, B24 chapter level, HOML). Nearest: PML2 15.3.9 Berkeley admissions, 19.2.4 selection bias, all statistical | **Lecture in 7 of 13 offerings**, never a parent unit: CS229 W25 L19 and L20; 10-701 L14 (2026), L19 (2024); 10-601 L14; 10-701 2021 L17 (FATE); CS231n L18 (human-centered AI); CS224n L16. None in CS229 2018 and 2026, CS189, Cornell, MIT |
| Interpretability | PML2 33 (own chapter); PML1 18.6; ESL 10.13 | CS224n L15; folded into CS231n L9 |
| Causality | PML2 36 (own chapter, in Action) plus 4.7 (structural causal models, inside the graphical-models chapter); PML1 3.1.4; GBC 15.3 | none |
| Deployment, MLOps | HOML 2 (launch, monitor, maintain), 19, App A; GBC 11, 12.1; PML2 19 (distribution shift, continual learning) | none; nearest CS231n L11 distributed training |

**Verdict on the previous finding:** "fairness has no canonical placement" is confirmed for textbooks and **partly overturned** for courses: it exists as a late standalone lecture whose neighbours differ every time.

## 8. Foundations

Confidence: high.

| Source | Probability | Statistics, decision | Linear algebra | Optimization | Information theory |
|---|---|---|---|---|---|
| PML1 | 2, 3 | 4, 5 | 7 | 8 | 6 |
| PML2 | 2 | 3 | assumed | 6 | 5 |
| GBC | 3 | 5 | 2 | 4, 8 | 3.13 |
| MML | 6 | inside 8 | 2, 3, 4 | 7 | none |
| PRML | 1.2, 2, App B | 1.3, 1.5 | App C | App D, E | 1.6 |
| B24 | 2, 3 | inside 4, 5 | appendix | 7 | none at chapter level |
| ESL | assumed | 2.4, 7, 8 | assumed | scattered | 7.8 |
| HOML | assumed | ch 1 | online notebooks only | 4, 11, App B | none |
| Courses | No course front-loads a math block: one or two named lectures (10-701 L5 to 6, CS189 L5 and L8) plus fragments at first use |

Three textbook styles: front-loaded part (PML1, PML2, GBC, MML), appendix plus intro chapter (PRML), assumed (ESL, HOML). Courses fold math in at point of use.

## 9. Where the field is growing

Confidence: medium; counts exact, "growing" inferred.

| Division | Old (PRML, ESL, GBC) | New (PML1, PML2, B24, HOML, 2025 to 2026 courses) |
|---|---|---|
| Attention, transformers, LLMs | none | PML1 15.4 to 15.6; B24 12 (50 pages); HOML 16; CS229 notes Part V; CS224n L5 to 13; 10-701 L13, L19 |
| Generative models (VAE, flows, diffusion, GAN) | GBC 20 only | PML2 Part IV (7 ch); B24 17 to 20; HOML 17; CS231n L13 to 14; CS229 notes Part V |
| Graph neural networks | none | PML1 23; PML2 30; B24 13 |
| Fewer labels (transfer, self-supervised) | GBC 7.6, 15.2 | PML1 19; PML2 32; CS231n L12 |
| Distribution shift, continual learning | none | PML2 19 |
| Fairness, interpretability, causality | ESL 10.13 | PML2 33, 36; fairness lectures in 7 of 13 course offerings |
| RL and decision making | none | PML2 34 to 36; HOML 18; every course |
| Deployment, scale | GBC 11, 12 | HOML 12, 13, 19 |

Dropped from new ToCs: Boltzmann machines and DBNs (GBC 20), RVMs (PRML 7.2), PRIM and MARS (ESL 9), SOMs (ESL 14.4); SVMs, GDA, naive Bayes, factor analysis, LQR dropped from CS229 lectures between 2018 and 2026. CMU 10-715 was replaced entirely by a rotating topics course (LLM speedruns, research agents) with no classical curriculum.

## 10. Teaching order versus reference order, and Géron Part I

Confidence: high for the mapping; "lifecycle" label is ours.

- Teaching order is a task spiral (regression, classification, NN) with cross-cutting ideas inserted at first need. Reference order (the CS229 notes, PML1) collects generalization and regularization once. Fast-path courses (10-701, MIT, CS229 W25) reach neural networks by lecture 6 to 11 and push SVMs, kernels, and theory to the end or out.
- **Implication (inference):** notes taken in teaching order scatter one concept across the places it was first met. A lookup tree should follow reference order; teaching order should be an overlay (an index or reading-order note), not the tree.
- Géron Part I: ch 1 to 2 lifecycle (frame, get data, explore, prepare, select and train, fine-tune, launch and monitor); ch 3 evaluation before training, the reverse of every theory book; ch 4 to 7 families by complexity; ch 8 to 9 unsupervised.

| HOML | PML1 | ESL |
|---|---|---|
| 1 overfitting, testing, validating | 1.2 to 1.4; 4.5.4, 4.5.5 | 2.1 to 2.3, 2.9, 7.10 |
| 2 scaling, categorical handling, pipelines | 1.5.3, 1.5.4 | none |
| 2 CV, grid search | 4.5.5, 5.4.3 | 7.10, 7.11 |
| 2 launch, monitor, maintain | none | none |
| 3 precision, recall, ROC | 5.1.3, 5.1.4 | 7 (indirect) |
| 4 linear regression, normal equation, GD variants | 11.2; 8.2, 8.4 | 3.2 |
| 4 ridge, lasso, elastic net, early stopping | 11.3, 11.4, 4.5.6 | 3.4, 3.8, 16.2 |
| 4 logistic, softmax | 10.2, 10.3 | 4.4 |
| 5 SVM, kernels, dual | 17.1, 17.3 | 12.2, 12.3, 5.8 |
| 6 trees, CART | 18.1 | 9.2 |
| 7 bagging, forests, AdaBoost, gradient boosting, stacking | 18.2 to 18.5 | 8.7, 15, 10.4, 10.10, 8.8 |
| 8 PCA, random projection, LLE | 20.1, 20.4.8 | 14.5, 14.9 |
| 9 k-means, DBSCAN, GMM, anomaly detection | 21.3, 21.4, 3.5 | 14.3, 6.8 |

In theory books but absent from HOML Part I: probability and MLE, decision theory, bias-variance as a theorem, Bayesian versions of each model. In HOML only: preprocessing pipelines, hyperparameter search tooling, launch and monitor.

## 11. Test questions a proposed tree must pass

1. Regularization sits in 4 to 5 places per book and is taught at lecture 2, 8, 13, or 22. One home, reachable from ridge, dropout, and tree depth without copies?
2. Every theory book front-loads probability, statistics, linear algebra, optimization; no course does. Is there a foundations area, so optimization is not trapped inside deep learning or linear models?
3. EM lives under optimization, mixtures, HMMs, and inference. One note reachable from all four?
4. PCA is filed under linear algebra, latent variables, dimensionality reduction, unsupervised, and beside autoencoders. Which wins, and are the rest links?
5. Deep learning is a chapter, a part, a book, or interleaved. Can one folder absorb B24's 20 chapters without moving anything?
6. All new divisions land in PML2's purpose axis. Are there homes for generation, discovery, and action outside "supervised"?
7. Kernels surface as smoothing, SVMs, RKHS, GPs, and attention. First-class, or buried under SVM?
8. Evaluation precedes models in HOML and follows them in ESL and PML1. Its own area?
9. No book has a fairness chapter; seven course offerings have a fairness lecture with a different neighbour each time. A home that is not a leaf under a method?
10. Deployment exists only in HOML and GBC 11. Does lifecycle have a home apart from theory?
11. Logistic regression is filed under classification, regression, GLMs, or MLE. One path without a family-versus-task choice at filing time?
12. Can the LLM lifecycle (pretraining, post-training, RLHF, RAG, agents) grow to a dozen notes without being wedged under neural networks?
13. RL is a separate end unit everywhere but RLHF, RLVR, PPO belong to LLMs too. A top-level sibling with LLM notes linking in?
14. Graphical models, EM, HMMs, learning theory are absent from 2026 fast-path courses but present in every reference text. Slots exist now?
15. Dropout, batchnorm, AdamW are taught inside the NN block or with optimization. Decided once where family-specific training tricks live?

## Verification

| Source | Read | URL and level |
|---|---|---|
| PML1 | yes | probml.github.io toc1.pdf; part, chapter, section, subsection |
| PML2 | yes | github probml/pml2-book toc2-long-2023-01-19.pdf; same depth |
| ESL | yes | hastie.su.domains contents.pdf; chapter, section, subsection |
| GBC | yes | deeplearningbook.org and contents/TOC.html; part, chapter, section |
| PRML | yes | Microsoft Research PDF; chapter, section, appendix |
| B24 | chapter level | archive.org snapshot of Springer book page; bishopbook.com solutions PDF for appendix citations |
| HOML | yes | archive.org snapshot of the O'Reilly page (parts, chapters, sections, appendices); handson-ml3 notebooks 01 to 09 cross-check |
| MML | yes | mml-book.github.io; part, chapter |
| CS229 | yes | syllabus-autumn2018.html; w24-index.html (Winter 2025); index.html-summer25; the Summer 2026 Google Sheet; main_notes.pdf (Aug 2026) |
| CS189 | yes | people.eecs.berkeley.edu/~jrs/189/ (Spring 2025) |
| 10-701 | **yes, gap closed** | cs.cmu.edu/~10701-s26/ (Spring 2026; schedule is inside index.html); cs.cmu.edu/~hchai2/courses/10701/ (2024); ~aarti/Class/10701_Spring21/lecs.html; ~epxing/Class/10701/lecture.html (2015) |
| 10-601, 10-715 | yes | ~mgormley/courses/10601/schedule.html (2026); ~10715-f18/lectures.shtml; ~10715/ (Fall 2026) |
| CS231n, CS224n | yes | cs231n.stanford.edu/schedule.html (Spring 2026); web.stanford.edu/class/cs224n/ (Winter 2026) |
| CS4780, 6.390 | yes | cs.cornell.edu/courses/cs4780/2018fa/lectures/; introml.mit.edu/fall26/calendar |

## Could not verify

- B24 section-level ToC and exact appendix titles (Springer front matter is behind a client challenge; the free reader is image-based). Their existence and subject matter are verified from the authors' solutions manual.
- Fairness in textbook body text (ToC keyword search only).
- Shalev-Shwartz and Ben-David (PDF returned an HTML stub).
- CS229 Winter 2025 TA lecture contents; CS229 2018 lecture 4 topic; CS231n lecture 18 contents.

## Sources

- https://probml.github.io/pml-book/toc1.pdf ; https://github.com/probml/pml2-book/raw/main/toc2-long-2023-01-19.pdf
- https://hastie.su.domains/ElemStatLearn/contents.pdf
- https://www.deeplearningbook.org/ ; https://www.deeplearningbook.org/contents/TOC.html
- https://www.microsoft.com/en-us/research/uploads/prod/2006/01/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf
- https://www.bishopbook.com/ ; https://www.bishopbook.com/Concepts_in_Deep_Learning_Solutions_v1.0.pdf ; https://web.archive.org/web/2025/https://link.springer.com/book/10.1007/978-3-031-45468-4
- https://web.archive.org/web/2024/https://www.oreilly.com/library/view/hands-on-machine-learning/9781098125967/ ; https://raw.githubusercontent.com/ageron/handson-ml3/main/index.ipynb
- https://mml-book.github.io/
- https://cs229.stanford.edu/syllabus-autumn2018.html ; https://cs229.stanford.edu/w24-index.html ; https://cs229.stanford.edu/ ; https://cs229.stanford.edu/main_notes.pdf
- https://people.eecs.berkeley.edu/~jrs/189/
- https://www.cs.cmu.edu/~10701-s26/ ; https://www.cs.cmu.edu/~hchai2/courses/10701/ ; https://www.cs.cmu.edu/~aarti/Class/10701_Spring21/lecs.html ; https://www.cs.cmu.edu/~epxing/Class/10701/lecture.html
- https://www.cs.cmu.edu/~mgormley/courses/10601/schedule.html ; https://www.cs.cmu.edu/~10715-f18/lectures.shtml ; https://www.cs.cmu.edu/~10715/
- https://cs231n.stanford.edu/schedule.html ; https://web.stanford.edu/class/cs224n/
- https://www.cs.cornell.edu/courses/cs4780/2018fa/lectures/ ; https://introml.mit.edu/fall26/calendar
- Failed: link.springer.com book page and front-matter PDF (client challenge); oreilly.com (403); cs.cmu.edu/~10701/ (bare directory); Shalev-Shwartz PDF (HTML stub)
