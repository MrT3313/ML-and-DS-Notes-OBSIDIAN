---
note_kind: concept
aliases:
  - machine learning applicability
  - ML applicability
  - when to use machine learning
  - when machine learning applies
  - is machine learning the right tool
  - good machine learning problem
  - machine learning problem fit
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
confidence: draft
---

## Definition

Machine learning applicability is the set of conditions a problem has to meet before machine learning is the right tool for it. Stating them as conditions rather than as advice is the point: the useful direction is negative, and a checkable list is what lets a problem be ruled out early, before anyone has collected data, bought hardware or hired for it.

## Formal statement

There are nine conditions and they are not of one kind. The first five are **necessary**: each one gates applicability on its own, and a problem that fails any single one of them is not a machine learning problem however well it scores on the other eight. The last four are **amplifiers**: a problem can fail all four and still be solvable by machine learning, but failing them is what makes the solution cost more than it returns. The two sets answer different questions. You check 1 through 5 to decide whether the approach can work at all. You check 6 through 9 to decide whether it is worth paying for. In [[DMLS Ch01 Overview of Machine Learning Systems|DMLS chapter 1]] the first five are the phrases the definition itself turns on, and the last four are the circumstances under which the approach especially pays.

**Necessary. Fail any one row and machine learning does not apply.**

| condition | what it requires | how it fails |
|---|---|---|
| 1. Learn | The system infers its behaviour from data rather than being told it. Something in the setup has the capacity to learn. | The behaviour is fully specified in advance by whoever builds it, so nothing is inferred and there is nothing for a learner to do. |
| 2. Complex patterns | Patterns exist in the data, and they are complex enough that learning them beats writing them down. | Two separate failures, and they are not the same defect. No pattern at all, as in successive rolls of a fair die. Or a pattern so simple that enumerating it is exact and cheap, as in mapping a zip code to its state. |
| 3. Existing data | Data is available, or it is possible to collect it. | No data, no access to any, and no path to generating it. Partial failure is [[Insufficient Training Data]], which is a matter of degree rather than a hard no. |
| 4. Predictions | The answer wanted is a prediction: an approximate guess at a quantity that is not known yet or not observed. | What is wanted is not a guess. An exact answer, a proof, a decision that has to be justified rule by rule, or a computation whose correct output is already determined by its inputs. |
| 5. Unseen data | The data the system meets in production shares its patterns with the data it trained on. | The production distribution differs from the training distribution, so what was learned does not carry over. This is [[Generalization]] restated as an entry requirement. |

**Amplifiers. These do not gate applicability. They decide whether the investment pays.**

| condition | what it requires | how it fails |
|---|---|---|
| 6. It is repetitive | The pattern recurs many times, so there are many examples of the same thing to learn from. | The pattern appears a handful of times. A person can often generalise from that few; an algorithm generally cannot, and the gap in how many examples each needs is the whole content of this row. |
| 7. Wrong predictions are cheap | An individual wrong prediction is absorbed without much damage. A bad recommendation is usually just not clicked. | Each mistake is expensive. This disqualifies nothing by itself: the test is the average, whether the benefit of the correct predictions outweighs the cost of the wrong ones over the whole population of predictions. |
| 8. It is at scale | The solution is used many times, so the up-front investment in data, compute, infrastructure and people is amortised across a large number of predictions. | One-off or low-volume use, where a fixed cost that is nontrivial no matter what is charged against a handful of predictions. |
| 9. The patterns keep changing | The patterns move, so a solution that can be retrained beats hand-written rules that go stale and have to be rewritten. | The patterns are stable. Hand-written rules stay correct, and they are cheaper to build, cheaper to keep, and easier to explain. |

Row 7 is the one most often read as a veto and is not one. The qualification is an expected value statement, which means it is a claim about a [[Utility Function]] or, in the sign convention this vault uses more often, a [[Cost Function]]: average benefit against average cost across every prediction the system will make, not the worst case taken on its own.

Row 6 is usually introduced by contrasting an algorithm with a child who recognises cats after seeing a few pictures. That contrast is an illustration, not a measured result. The research it gestures at is Lake, Salakhutdinov and Tenenbaum's 2015 work on one-shot concept learning (*Science* 350(6266), 1332 to 1338), where the concepts are handwritten characters from unfamiliar alphabets and the comparison is a specific model against human participants on a specific benchmark. Treat the cat as the analogy it is.

### Testing the pattern condition

Condition 2 carries two clauses and a standing caveat, and each of the three is checked differently.

**No pattern.** Successive rolls of a fair die are independent and identically distributed, every face at probability $1/6$. So the distribution of the next roll conditioned on the entire history of previous rolls is the same as its unconditional distribution, and no function of anything observable predicts the next outcome better than the constant guess does. This is true by construction rather than by observation: independence and fairness are assumptions of the setup, not findings from the rolls. Note what is and is not absent. The uniform distribution is itself a perfectly learnable piece of structure. What does not exist is any dependence of the outcome on an input, which is the only kind of structure a predictor can use.

**A pattern too simple to learn.** A zip code determines its state exactly, and the mapping is a finite enumeration that can be written down once and is then never wrong. A lookup table is the correct engineering answer, and a learned approximation of a relation that is already exact is strictly worse on every axis: less accurate, more expensive, and harder to check. This row fails on the word *complex*, not on the word *pattern*.

**Failure to predict is not evidence of absence.** A model that cannot make reasonable predictions has established nothing about whether a pattern exists. The wrong features, too few instances, or the wrong model class each produce the same negative result that a genuinely patternless problem does. A failed attempt bounds what was tried, not what is there.

### Routes around the data condition

Condition 3 asks for data that exists or can be collected, and it is the one requirement with recognised ways around it. A model can ship without having been trained on anything and learn from production traffic as it arrives, which is [[Continual Learning]]. A system can answer for a task it was never trained on, which is [[Zero-Shot Learning]]. And a product can launch serving predictions made by people rather than by a model, on the expectation that the data those predictions generate will train the model later, a fake it until you make it strategy that satisfies condition 3 prospectively rather than at launch.

## Where it is used

These are conditions on [[Machine Learning]] as a whole, checked before a project starts rather than during it. Condition 5 is the generalization assumption stated as an entry requirement, so [[Generalization]] is the same claim seen from the other end, and [[Insufficient Training Data]] is what condition 3 looks like when it is met only partly. The two exceptions to condition 3 are [[Continual Learning]], where the model ships untrained and learns from live traffic, and [[Zero-Shot Learning]], where predictions are made for a task that was never trained for. Condition 6 turns on the gap between the handful of examples a person needs and the many an algorithm needs, which is [[Few-Shot Learning]] named from the human side. Condition 8's up-front investment is spent on building a [[Machine Learning System]], and it is paid where that system runs, in [[Production Machine Learning]]. Condition 9 has a mirror image after deployment: constantly changing patterns are the reason to prefer a learned solution over hand-written rules, and the same changing patterns are [[Model Rot]], which is what degrades that solution once it has shipped. Condition 7's expected value qualification is a statement about a [[Utility Function]] or a [[Cost Function]], averaged across predictions rather than evaluated one at a time.
