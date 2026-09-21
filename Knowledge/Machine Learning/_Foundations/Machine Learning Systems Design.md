---
note_kind: concept
aliases:
  - machine learning systems design
  - ML systems design
  - machine learning system design
  - ML system design
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

Machine learning systems design is the activity of considering a system whole, so that its components and everyone with a stake in it together satisfy the objectives and requirements that were actually set. It is an activity rather than an artifact: it fixes what the system has to do, and it checks that the thing built does it.

Two neighbouring notes answer different questions and are worth separating on arrival. [[Machine Learning System]] is what the thing is made of, the code, the training data and the artifact fitted from the two, while this note is the activity of deciding what that thing has to do and of checking that it does. [[Systems Architecture]] is the neighbouring domain, about how any software system is arranged, how many machines run it and who owns them, while this is about whether one particular learned system meets the objectives and requirements its stakeholders actually set.

Four requirements stand behind the phrase "the requirements that were actually set": reliability, scalability, maintainability and adaptability.

## Formal statement

| Requirement | What it asks of any software system | What a system fitted from data adds |
| --- | --- | --- |
| **Reliability** | That it go on performing the correct function at the intended level under adversity: [[Reliability]]. | A learned system can fail the requirement while every component keeps answering and nothing crashes, so the ordinary signal that something is wrong is simply absent. [[Machine Learning System]] carries the comparison against software whose behaviour was written by hand, and the failure has its own name, [[Model Rot]]. |
| **Scalability** | That it go on doing so as the load on it grows: [[Scalability]]. | Load grows on a third axis here, beside complexity and traffic volume, which is the number of models. One model detects trending hashtags, then another filters unsafe content, then another filters bot-generated posts, and in the limit there is one model per customer, so the model count $m$ equals the customer count $n$, $m = n$. Artifact management is the second half of that growth. Monitoring, retraining and reproducing models requires automation as soon as there is at least one model in production, $m \geq 1$, and the claim is the threshold rather than the merit of automation: it is one model, not many. |
| **Maintainability** | That the people who have to keep it working are able to: [[Maintainability]]. | What is versioned is not only the source. [[Machine Learning System]] states which three things the deployed behaviour is jointly a function of and why each has to be pinned on its own. The human half of the requirement is the other clause: models should be reproducible enough that contributors who arrive after the original authors have left have the context to build on the work. |
| **Adaptability** | The nearest general requirement is evolvability, one of the three design principles under maintainability: that engineers can keep changing the system in future, adapting it as the requirements on it change. | The system should be able to adapt to shifting data distributions and to changing business requirements, which asks for two capacities: discovering where performance can be improved, and allowing updates without interrupting service. The argument for it is that a machine learning system is part code and part data, and data can change quickly, so the system has to be able to evolve quickly. Evolvability already covers changing requirements, which any long-lived software faces. What adaptability adds beyond it is the shifting distribution, a half that cannot be stated without a fitted model in the sentence: it is [[Model Rot]] under its own name, and [[Continual Learning]] is the standing response to it. The two capacities named are monitoring and deployment mechanics, and neither has a note here yet, which is why this row states a requirement and not the mechanism that meets it. |

These are requirements and not properties a system has or lacks, and that is the falsifiable part of the table. No design satisfies all four in general, because the four trade against one another. The concrete instance is the last row against the first: adaptability asks for frequent updates, and reliability asks for a system that does not change under you, so the update cadence is a number negotiated between the two rather than a number maximized.

## Where it is used

[[MLOps]] is the half of this that begins at deployment, and it is the note that named the subject before the subject had a note. [[Machine Learning Project Lifecycle]] is the shape of the work this is the design of, six phases that close on themselves, which is why a design here is revised rather than completed. [[Machine Learning System]] is the object designed, and its three parts are what the four requirements above are requirements on.

[[Machine Learning Applicability]] is the gate in front of all of it, since a problem that fails those conditions is not designed, it is declined. [[Production Machine Learning]] is the setting in which several parties each want something different from one model, which is why "the requirements that were actually set" is plural and why meeting them is a conjunction rather than a maximum. [[Business Objective]] is the first thing this activity fixes, since the requirements a system is finally checked against are set from it rather than from the model.
