---
note_kind: concept
aliases:
  - self-host
  - self-hosted
  - self-hosting
  - on premises
  - on-premises
  - on-prem
  - on premise
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Self-hosting is deploying and operating software on machines you control, whether those machines sit in your own building or are rented as bare virtual servers from someone else. What makes it self-hosting is not where the metal is but who is on call for it.

It is also the middle answer to "should you build it or buy it". Building means writing the system; buying means renting a finished service. In between is off-the-shelf software, open source or commercial, that somebody else wrote and you deploy yourself, which gets you the code without getting you the vendor.

## VS

[[Cloud Computing]] states the deciding conditions: whether you already know how to operate the system, and whether your load is predictable. This side of the trade adds what a vendor cannot hand over.

- **Cost.** If you have experience setting up and operating the system you need, and load is predictable, it is often cheaper to buy your own machines and run the software on them yourself. "Cheaper" is a claim about the whole bill, hardware plus the hours of the people who keep it alive, so it holds only while the experience clause holds. Experience you do not have is bought with time before it is bought with money.
- **Fit.** You can configure and tune the system to your specific workload. A managed service is tuned for the average of everybody's workload, which is a different target and is not adjustable on request. The freedom is also an obligation: nothing tunes itself, so the fit is only real if somebody does the work.
- **Custody.** The data never leaves machines you control, so the privacy and security questions are answered by your own controls rather than by a contract with a vendor.

## Formal statement

Self-hosting is defined by the responsibilities it leaves with you, and the list is exactly what a managed service takes away. Each row is a standing obligation rather than a one-time cost, which is why a deployment decision made on the price of hardware alone is being made on the wrong number.

| responsibility | what holding it means |
|---|---|
| hardware | buying machines, replacing failed ones, and eating the depreciation |
| capacity | sizing for the peak you expect, in advance, and owning the gap between that peak and the average |
| upgrades | choosing when to take a new version and carrying the risk of the upgrade itself |
| durability | backups, restore drills, and proving the restore works before you need it |
| availability | someone on call, monitoring that pages them, and a diagnosis path into the machine |

## Where it is used

[[Cloud Computing]] is the alternative, and the deployment question is only decidable with both sides in view. [[DevOps]] is where the burden in the table above actually lands: self-hosting means the operational work is yours, and the automation, the ephemeral instances and the incident reviews are how a team of a workable size absorbs it. [[Cloud-Native Architecture]] is the claim that some systems designed for the other answer do things a self-hosted system cannot, and it carries the product-level comparison of what is available on each side.
