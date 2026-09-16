---
note_kind: concept
aliases:
  - triangle inequality
  - triangle inequalities
  - Minkowski inequality
  - Minkowski's inequality
  - Minkowski inequalities
  - subadditivity
  - subadditive
  - reverse triangle inequality
up: "[[Mathematics]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

Adding two vectors never yields something longer than their two lengths added separately. Read as distance, it says a detour through a third point is never shorter than going direct, which is the statement about the sides of a triangle the name comes from.

## Formal statement

$$\lVert \mathbf{u} + \mathbf{v} \rVert \leq \lVert \mathbf{u} \rVert + \lVert \mathbf{v} \rVert$$

This is one of exactly three axioms a function $\lVert \cdot \rVert$ must satisfy to be called a norm. The other two are absolute homogeneity, $\lVert \alpha \mathbf{v} \rVert = |\alpha| \, \lVert \mathbf{v} \rVert$, and positive definiteness, $\lVert \mathbf{v} \rVert = 0$ only for $\mathbf{v} = \mathbf{0}$.

A norm induces a distance by $d(\mathbf{u}, \mathbf{v}) = \lVert \mathbf{u} - \mathbf{v} \rVert$, under which the same axiom reads

$$d(\mathbf{u}, \mathbf{w}) \leq d(\mathbf{u}, \mathbf{v}) + d(\mathbf{v}, \mathbf{w})$$

**Minkowski's inequality** is the theorem that the $\ell_p$ formula satisfies it, and it holds precisely for $p \geq 1$. Below $1$ it fails, and one pair shows it. Take $\mathbf{u} = (1, 0)$, $\mathbf{v} = (0, 1)$ at $p = 1/2$. Each vector has $\lVert \cdot \rVert_{1/2} = (1^{1/2} + 0^{1/2})^{2} = 1$, so the right side is $2$. But $\mathbf{u} + \mathbf{v} = (1, 1)$ gives $(1^{1/2} + 1^{1/2})^{2} = 2^{2} = 4$, twice the bound it was supposed to respect.

**Equality.** For $p > 1$ equality requires $\mathbf{u}$ and $\mathbf{v}$ to be positively linearly dependent: one is a non-negative multiple of the other, or one is zero. Every other pair is strict. At $p = 1$ the condition is far weaker, holding whenever $u_i v_i \geq 0$ for every coordinate, so any two vectors in the same orthant achieve it.

**Reverse form.** Writing $\mathbf{u} = (\mathbf{u} - \mathbf{v}) + \mathbf{v}$ and repeating with the roles swapped gives

$$\big| \, \lVert \mathbf{u} \rVert - \lVert \mathbf{v} \rVert \, \big| \leq \lVert \mathbf{u} - \mathbf{v} \rVert$$

which is the form proofs about stability and continuity actually use: a small change to a vector can only change its length by as much as the change itself.

**Convexity.** For $\lambda \in [0, 1]$, subadditivity followed by homogeneity gives $\lVert \lambda \mathbf{u} + (1 - \lambda) \mathbf{v} \rVert \leq \lambda \lVert \mathbf{u} \rVert + (1 - \lambda) \lVert \mathbf{v} \rVert$. Those two axioms together are exactly what makes a norm a convex function, so its sublevel sets, the unit ball among them, are convex sets.

## Where it is used

[[Lp Norm]] is restricted to $p \geq 1$ by this inequality and nothing else, and its remark that $p < 1$ unit balls are non-convex is the convexity argument above running backwards. The convexity it grants is why a [[Cost Function]] built as a norm penalty has one global optimum rather than a landscape of local ones, and why $p < 1$ penalties forfeit that guarantee. [[Root Mean Squared Error]] and [[Mean Absolute Error]] are each a norm of the residual vector up to a constant, so both inherit it and are convex in the residuals. In the distance form it is what lets [[Instance-Based Learning]] and [[Clustering]] prune: bounding the distance to a point you have not measured, using distances you already computed, is how ball trees and accelerated k-means avoid comparing every pair.
