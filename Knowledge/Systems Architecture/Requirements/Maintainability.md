---
note_kind: concept
aliases:
  - maintainability
  - maintainable
up: "[[Systems Architecture]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

Most of what a piece of software costs is not spent building it but keeping it running and changing it afterwards, so maintainability is the requirement that the people who work with the system later, including people who were not there when it was built, can do so without misery. Two parts of it are structural rather than a matter of attitude: workloads and infrastructure are set up so that different contributors can work using tools they are comfortable with, and the code is documented.

## Formal statement

The requirement is a sentiment until it is split, and the split that makes it checkable is into three design principles, each of which is a separate thing to build for and a separate thing to fail at.

- **Operability.** Make it easy for the people operating the system to keep it running smoothly. The test is on routine work: how much of what the operators do every week is something a machine could have done, and how much of what they need to know during an incident is visible without asking the person who wrote it.
- **Simplicity.** Make it easy for a new engineer to understand the system, by removing accidental complexity. The test is on time to comprehension, not on the size of the codebase, and certainly not on the simplicity of the user interface, which is a different subject with the same adjective.
- **Evolvability.** Make it easy for engineers to change the system later as requirements change. The test is on unanticipated change: a system that absorbs the modifications its designers foresaw has demonstrated nothing, since those were designed in.

**Accidental complexity** is the term that makes the second of these precise, and it is precise because it is defined by reference to the problem rather than to anyone's taste. Complexity is accidental when it is not inherent in the problem the software solves, as the users see that problem, and arises only from the implementation. The vocabulary is Brooks's, borrowed in turn from Aristotle:

> All software construction involves essential tasks, the fashioning of the complex conceptual structures that compose the abstract software entity, and accidental tasks, the representation of these abstract entities in programming languages and the mapping of these onto machine languages within space and speed constraints.
> ~ Brooks, *No Silver Bullet: Essence and Accident in Software Engineering*, 1986

That definition yields a test rather than an opinion: **would this piece of complexity survive a reimplementation against the same requirements?** If a second team, given the same user-visible problem and free to choose its own tools, would have to build the thing anyway, it is essential. If it would not, it is accidental and it is a defect that happens to compile. Abstraction is the tool that removes it, by naming a piece of the implementation well enough that callers no longer have to hold it in their heads.

The line is not uncontested, and the contested part is worth locating exactly. What survives is the definition. What does not is Brooks's estimate of the proportions: he put the accidental fraction at less than a tenth of all effort, and Dan Luu's "Against Essential and Accidental Complexity" (2020) argues that the bound is unsupported, that what counts as inherent in a problem moves as tools improve, and that in practice non-trivial work is dominated by accidental complexity rather than the reverse. The test above does not depend on the proportions, so it holds either way.

### The three are traded, not summed

They are distinct requirements, and one is routinely bought with another. [[Microservices]] buys evolvability, since each service can be released on its own schedule, and pays for it in operability multiplied by the number of services, since every one of them now needs its own deployment, logging and monitoring. An abstraction introduced for simplicity adds a layer that somebody has to see through when the system misbehaves, which is operability spent. Automation added for operability removes the routine work that kept the operators fluent in how the system behaves, so the rare manual intervention is attempted by people with less practice than they used to have (Bainbridge's "Ironies of Automation", 1983). None of these trades is a mistake. Making one without noticing which of the three is being spent is.

### Why the requirement outranks the build

Write $m$ for the share of a system's whole-life cost that falls after the first release,

$$m = \frac{\text{maintenance cost}}{\text{total lifecycle cost}}$$

Glass reports $m$ between 40% and 80%, averaging about 60%, over the studies he surveys (*Facts and Fallacies of Software Engineering*, 2002, fact 41). The build is the minority of the spend, so a decision is being evaluated on the wrong fraction whenever it is evaluated on how long it takes to write. The arithmetic of a shortcut follows directly: one that saves 40 hours now and costs one hour a week is behind after 40 weeks and keeps losing after that, and a system of any age is well past week 40.

## Where it is used

[[DevOps]] is the practice organized around the operability half of this, and it already carries the part that is about people rather than machines, preserving what the organization knows about the system as individual people come and go. [[Microservices]] is the standing example of the trade above, evolvability bought with operability, which is why the architecture only pays for an organization that has the operational practices in place first. [[Cloud Computing]] moves part of the operability burden onto a vendor, and the control table there is the price of that move: the work that leaves is the work you can no longer do yourself when it goes wrong.

[[Machine Learning Systems Design]] is what this requirement asks of a system whose behaviour is fitted from data rather than written by hand, where the thing a later contributor has to be able to pick up is not only the code.
