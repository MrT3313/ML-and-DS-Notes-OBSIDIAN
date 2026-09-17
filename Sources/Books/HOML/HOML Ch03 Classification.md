---
note_kind: source
medium: book
chapter: 3
up: "[[HOML]]"
url: https://github.com/ageron/handson-ml3/blob/main/03_classification.ipynb
aliases:
  - HOML Chapter 3
  - HOML Classification
---

## Thesis

Chapter 3 is an argument that one number does not evaluate a classifier, made on a target chosen so the number lies. Relabelling MNIST into a 5-versus-rest detector leaves about nine percent of the training set positive, and the chapter's linear detector cross-validates to roughly 95 percent accuracy on three folds while a constant "not a 5" predictor, which has looked at no pixel at all, scores about 91 percent on the same folds. So accuracy goes and the confusion matrix replaces it: the full table of predicted labels against true ones, out of which accuracy is only the diagonal over the total and out of which precision and recall take the two complementary slices that survive an imbalanced target, one conditioned on the column the model chose and one on the row the data fixed. The deeper point comes next, and it is what makes the pair inseparable. Precision and recall are not two properties a model has, they are two readings taken at one setting of a decision threshold that is not fitted and carries no information about the data, so raising the cut point drives recall down monotonically while precision trends up, and a figure quoted for either alone was produced by a choice rather than measured off a model. That is what my own rule is getting at: anytime someone says "Let's reach 99% precision", you should ask "At what recall?" The chapter answers its own version of that question in numbers, buying 90 percent precision on the 5-detector at the cost of half the fives. The second movement turns the same sceptical eye on capability rather than on scores. Multiclass, multilabel and multioutput look like things scikit-learn estimators can do, and mostly they are binary classifiers inside meta-estimators, one per class or one per pair or one per label, applied silently on your behalf unless you name a different strategy, which means the decomposition that decided how the model was actually fitted is a thing you inherit rather than a thing you chose.

## Extracted

The task and its varieties:
- [[Binary Classification]]
- [[Multiclass Classification]]
- [[Multilabel Classification]]
- [[Multioutput Classification]]

The classifiers:
- [[Stochastic Gradient Descent Classifier]]
- [[Baseline Model]]

Binary classifiers wrapped into the other varieties:
- [[One-versus-Rest]]
- [[One-versus-One]]
- [[Classifier Chain]]

Reading the errors:
- [[Confusion Matrix]]
- [[Error Analysis]]
- [[Accuracy]]
- [[Precision]]
- [[Recall]]
- [[F1 Score]]

Sweeping the threshold:
- [[Precision-Recall Tradeoff]]
- [[ROC Curve]]

The data:
- [[Class Imbalance]]
- [[Data Augmentation]]
- [[MNIST]]

Amended, not produced, by this chapter:
- [[Cross-Validation]]
- [[Scikit-Learn Estimator API]]
- [[Ensemble Learning]]
- [[Skewed Data]]

Corrected by this chapter:
- [[Performance Measure]]
- [[Logistic Regression]]
- [[Classification]]

## Open questions

- `Objectives/Metrics/` now holds nine notes, two from chapter 2 for regression and seven from this chapter for classification, and more measures are coming: chapter 8 brings the reconstruction and explained-variance readings, chapter 9 brings the clustering scores, chapter 10 brings the losses that neural networks are trained against. On what trigger does the folder split, and on which axis? By task, which would put regression, classification and clustering in separate buckets and leave each thin for several chapters, or by shape, a scalar read at one operating point against a curve swept over a whole family of them, which would put [[Precision-Recall Tradeoff]] and [[ROC Curve]] together and everything else in the other bucket. Deferred deliberately rather than argued now, so the trigger is the thing to decide first.
- The chapter fits three classifiers this vault cannot explain. `SVC` is chapter 5's and the random forest is chapter 7's, so those two gaps have dates on them. k-nearest neighbours does not: the chapter uses it for the multilabel example, for the multioutput denoising example, and again in exercises 1 and 2 as the model being tuned and then retrained, and no HOML chapter takes it as its subject. The gap is already visible in the graph rather than only in this list, since [[Instance-Based Learning]] calls it the canonical method through a `[[K-Nearest Neighbours]]` link that resolves to nothing. Which chapter brings it, or is this the first note the vault has to write outside a chapter?
- Whether the printed chapter raises data augmentation in its own error-analysis prose is unverified. In the notebook it appears only as exercise 2 and its solution, headed `## 2. Data Augmentation`, and the Error Analysis section there carries no augmentation text at all, so the connection between the confusions found and the remedy proposed is one the chapter makes across its own sections rather than in a single paragraph. The book text is paywalled, so this needs checking against the printed error-analysis section. Nothing depends on the answer, since [[Data Augmentation]] is written either way; what moves is only how much weight the source is recorded as giving it.
- [[Home]] uses `## Domains` where the Quality Bar says a hub uses `## Areas`, and [[Machine Learning]] uses `## Areas`. Those are the only two hubs in the vault, so the Quality Bar's own tie-break, that the notes win when they disagree with the document, cannot decide a one-to-one split. Which heading is right? `Areas` is what the rule names and what the lower hub does; `Domains` is arguably the truer word for what `Home` actually lists, since `Machine Learning` and `Mathematics` are fields rather than folders of methods. Settling it means either changing a hub or changing the rule, and one of those two has to happen before a third hub exists and makes the majority accidental. This is a question about the vault rather than about the chapter, which chapter 2 already established as a legitimate thing to record here.
- Looking back: [[HOML Ch01 The Machine Learning Landscape]] asked whether the learning rate of an online system and the step size of gradient descent are the same knob, and this chapter does not settle it. It fits a [[Stochastic Gradient Descent Classifier]] and takes the optimizer that names the estimator entirely as given, never covering gradient descent itself, so the question passed to chapter 4, which has since settled it: they are one knob, a squared-error stochastic update on one instance being an exponential moving average whose smoothing factor is twice the step size, so the same $\eta$ sets how far a step travels and how fast the model forgets. The derivation is in [[Learning Rate]] and the settlement is recorded in [[HOML Ch04 Training Models]]. Chapter 3 settles none of [[HOML Ch02 End-to-End Machine Learning Project]]'s open questions either. The ensembles gap is touched and not closed, since a random forest appears here as a comparison case and [[Ensemble Learning]] still holds no mechanism. The MLOps gap, the feature extraction gap and the question of whether to split `Data/Preprocessing/` are untouched.
