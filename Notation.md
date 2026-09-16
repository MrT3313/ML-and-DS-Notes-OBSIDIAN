---
note_kind: meta
title: "Notation"
---

# Notation

- _lower case italic_ $\rightarrow$ scalar values & function names
- **lower case bold** $\rightarrow$ vectors
- **UPPER CASE BOLD** $\rightarrow$ matricies

| term / symbol                        | desc                                                                                                                                                                                                  |
|--------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| $m$                                  | number of instances                                                                                                                                                                                   |
| $\textbf{x}^i$                       | vector of all feature values (excluding the label) of the $i^{th}$ instance in the dataset                                                                                                            |
| $y^i$                                | label / desired output of the $x^i$ instance in the dataset                                                                                                                                           |
| $\mathbf{X}$                         | a matrix containing all the features values (excluding labels) of all instances in the dataset. 1 row per $i^{th}$ instance $=$ transpose of $\mathbf{x}^{(i)}$ $\rightarrow$ $(\mathbf{x}^{(i)})^T$  |
| $h$                                  | prediction function / _hypothesis_ $\rightarrow$ when given instance vector $\mathbf{x}^{(1)}$ its predicted output is $\hat{y}^{(i)} = h(\mathbf{x}^{(i)})$                                          |
| $\mathbf{u}, \mathbf{v}, \mathbf{w}$ | arbitrary vectors in a purely mathematical statement, no machine learning role $\rightarrow$ distinct from $\mathbf{x}^{i}$, which is always an instance's feature vector                             |
