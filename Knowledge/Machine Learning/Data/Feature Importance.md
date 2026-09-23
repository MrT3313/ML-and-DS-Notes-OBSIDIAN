---
note_kind: concept
aliases:
  - feature importance
  - feature importances
  - feature_importances_
  - permutation importance
  - permutation_importance
  - ablation study
  - ablation studies
  - ablation
  - impurity-based importance
  - SHAP
  - SHAP values
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

The importance of a feature is how much worse the model gets without it. Take the column away, or destroy what is in it, score the model again, and the deterioration is the measurement. The same procedure runs on a set of columns rather than one, which is the version that matters when columns come in groups that carry the same thing.

Two properties of the measurement are worth stating at the front, because both are easy to forget once a ranking is on the screen. It is a reading taken on **a particular fitted model against a particular scoring set**, not a property the column has on its own, so two models fitted to the same table can rank the same columns differently and both be right. And the readings concentrate: in practice a small number of columns accounts for most of the total importance and the rest of the ranking is close to flat, which is what makes the measurement worth taking at all, since it says where to look rather than merely ordering everything.

## Formal statement

Write $\mathcal{L}$ for risk estimated on a scoring set, $h$ for the model fitted with every column available, and $h_{-j}$ for the same model with feature $j$ removed. The importance of $j$ is the increase in risk,

$$I_j \;=\; \mathcal{L}\big(h_{-j}\big) - \mathcal{L}(h)$$

and the whole content of the subject is in what "removed" is taken to mean. There are two standard answers and **they measure different quantities**, which is the distinction the plain-English phrasing above hides.

- **Refit without the column.** $h_{-j}$ is a new model fitted on the other $n-1$ columns. This is the ablation version, and it measures what the column is worth *to the problem*, given that the fitting procedure is allowed to adapt to its absence.
- **Permute the column in held-out data.** $h$ is left exactly as it was fitted, and the values in column $j$ are shuffled among the rows of the scoring set. This is the permutation version, and it measures what *this fitted model* loses when the column stops carrying information. The model was still fitted with column $j$ present, so its parameters are still spending capacity on it, and nothing about it is re-optimized.

A column can therefore score high under one and low under the other, and the gap is informative rather than a discrepancy. The clearest case is a column the fit leans on heavily but that a second column could have supplied: permuting it hurts, because the fitted parameters have nowhere else to look, while refitting without it costs almost nothing, because the refit simply uses the other column. The costs differ too, and by more than a constant: ablation is one fit per feature, so $n + 1$ fits for a full table and $2^{n}$ if every subset is wanted, while permutation is one scoring pass per shuffle and no fits at all.

### Ablation

Refit on the reduced column set and compare. As a measurement it is the most direct reading of $I_j$ there is, because $h_{-j}$ really is a model that never saw the column, which is the counterfactual the definition names. Its price is the refit, and the price is what keeps it from being the default: on a table wide enough for the question to be interesting, $n$ refits is not a thing you run casually, and it is also noisy, since two fits of a stochastic learner differ from each other by an amount that can be comparable to $I_j$ for a middling column. Repeating each refit under several seeds and comparing the spread of $I_j$ against the spread of $\mathcal{L}(h)$ itself is what makes the number mean something.

Run deliberately rather than read off a fitted model, this is an **ablation study**: choose a feature or a set of features, take it out on purpose, refit, and record how far the score moved. It is one experiment and not two. Whether the motive is to measure what a column is worth or to find out why it is worth so much, the procedure is identical and only the question changes.

The second of those motives is the one [[Data Leakage]] points here for. A column whose removal costs a great deal is either genuinely important or is leaking the answer, and the removal experiment cannot tell those apart: both produce a large $I_j$, and both produce it for the same reason, that the model was relying on the column. So a large $I_j$ on a column whose meaning does not obviously justify it is a trigger to go and find out how that column is generated, not a finding. The check that settles it is [[Data Leakage]]'s recomputation, rebuilding the column from what production would actually hold at prediction time and comparing it against what the training file contains.

### Permutation

Leave the fitted model alone and break the column instead. Shuffling the values of $x_j$ among the rows of the scoring set destroys the association between that column and the target while leaving the column's own marginal distribution exactly as it was, so what the score loses is attributable to the association rather than to the column having gone strange.

The idea is Breiman's, in *Random Forests* (*Machine Learning* 45(1), 2001), where it runs on the out-of-bag rows. After each tree is built, the values of variable $j$ are permuted among that tree's out-of-bag cases and those cases are run down the tree again; at the end of the forest, the plurality of out-of-bag votes for each instance with variable $j$ noised up is compared against the true label, and what is reported is the percent increase in misclassification rate against the rate with every variable intact. Worth keeping straight: the aggregation there is one forest-level rate computed from the voted predictions, and the familiar per-tree-drop-then-average form belongs to the later Breiman and Cutler manual rather than to the paper, as does the impurity-based measure, which the 2001 paper does not contain at all.

Fisher, Rudin and Dominici (*JMLR* 20(177), 2019) take the operation off the random forest and make it model-agnostic, permuting the inputs to the model as a whole rather than to each ensemble member, and give it the name **model reliance**. Their definition is a ratio rather than a difference. Let $Z^{(a)}$ and $Z^{(b)}$ be independent copies of an instance, take the target and the other columns from $Z^{(b)}$ and the column of interest from $Z^{(a)}$, and write $e_{\text{switch}}(f)$ for the resulting expected loss against $e_{\text{orig}}(f)$ for the expected loss with nothing switched. Then

$$\mathrm{MR}(f) \;=\; \frac{e_{\text{switch}}(f)}{e_{\text{orig}}(f)}$$

so $\mathrm{MR} = 1$ is no reliance and $\mathrm{MR} = 2$ says the loss doubles when the column is scrambled. A difference form is available and they treat it as the alternative rather than the definition. Their empirical version averages over *every* pairing of rows rather than over one random permutation, which is what makes it unbiased, and their model class reliance goes one step further by taking the range of $\mathrm{MR}$ over every model whose loss is within $\epsilon$ of the best, so it says whether *any* well-performing model of that class must lean on the column. That last quantity does not depend on which fitting algorithm was used, which is precisely what a single fitted model's importance cannot tell you.

The estimator in ordinary use averages over repeats instead, because one shuffle is one draw and the number moves. Let $s$ be the reference score of the fitted model on the held-out set, and $s_{k,j}$ the score after the $k$-th independent shuffle of column $j$. With $K$ repeats,

$$\hat{I}_j \;=\; s \;-\; \frac{1}{K}\sum_{k=1}^{K} s_{k,j}$$

and the spread of the $K$ values is reported alongside the mean, since an importance smaller than its own standard deviation across shuffles is not a measurement of anything. The sign convention follows whether $s$ is a score to maximize or a loss to minimize; written as above with $s$ a score, a useful column gives $\hat{I}_j > 0$.

That formula is implemented exactly as written. In scikit-learn 1.6, `sklearn.inspection.permutation_importance(estimator, X, y, ...)` takes an already-fitted estimator, computes the reference score $s$, then for each column and each repeat shuffles that column and rescores, and returns `importances_mean`, `importances_std` and the raw $K \times n$ grid. `n_repeats` is the $K$ above and **defaults to 5**, which is worth knowing because the library's own examples pass $10$ explicitly and the two numbers get confused. `X` may be the training set or a held-out set, and held out is the point: scored on training rows the measurement inherits the same optimism as any training-set reading.

Two cautions travel with it and neither is optional. **Shuffling manufactures rows that could not exist.** A permuted column is recombined with the other columns at random, so the scoring set fills with combinations the data never contains, and the model is then being evaluated outside the region it was fitted on. Part of the score drop is the column mattering and part of it is the model extrapolating badly, and the two are not separated by anything in the procedure. This is worst exactly where the columns are most dependent on each other. **And correlated columns split their importance.** Two columns carrying the same signal each permute to a small drop, because the model reads what it needs off the other one, so both look unimportant while the pair is essential. The remedy is to permute the group together, which is the set-valued version of $I_j$ the definition already allows for.

### Shapley values

The removal experiments above take a column out of the full set. The Shapley value asks the same question over every subset at once, which is what makes it the principled answer rather than one more heuristic. Let $F$ be the set of features and $v(S)$ the value of a coalition $S \subseteq F$, ex the model's performance using only the columns in $S$. The Shapley value of feature $j$ is

$$\phi_j \;=\; \sum_{S \subseteq F \setminus \{j\}} \frac{|S|!\,\big(|F| - |S| - 1\big)!}{|F|!}\,\big[v(S \cup \{j\}) - v(S)\big]$$

(Shapley, *A Value for n-Person Games*, 1953). The bracket is the marginal contribution of $j$ to the coalition $S$, and the weight in front makes the whole thing an average over **orderings of the features, not over subsets**. Read the weight that way and it stops looking arbitrary: line the $|F|$ features up in a random order and ask what $j$ adds when it arrives. The number of orderings in which exactly the set $S$ has already arrived is $|S|!$ ways to arrange those before $j$ times $(|F| - |S| - 1)!$ ways to arrange those after, over $|F|!$ orderings in all, which is the coefficient exactly. So $\phi_j$ is the plain average of $j$'s marginal contribution over all $|F|!$ arrival orders, collected by subset.

That averaging is why the exact computation is out of reach. The sum runs over every subset of $F \setminus \{j\}$, of which there are $2^{|F| - 1}$, and each term needs $v$ evaluated twice, so the work doubles with every column added. Twenty columns is half a million terms for one feature; a hundred columns is not a number anyone computes. Everything practical is therefore an approximation, and the two kinds are worth telling apart: **sampling** methods estimate the average from a random subset of the orderings and are model-agnostic, while **model-specific** methods exploit the structure of one model family to get the exact answer in polynomial time.

What turns this from a game-theory object into a feature importance is the choice of $v$, and that choice is the content of Lundberg and Lee's SHAP (*NeurIPS* 2017). A trained model cannot be evaluated on a subset of its inputs, since it demands all of them, so they take $v(S)$ to be the model's output conditioned on the columns in $S$ being the only ones known,

$$v(S) \;=\; \mathbb{E}\big[f(\mathbf{x}) \mid \mathbf{x}_S\big]$$

and a SHAP value is then the Shapley value of that conditional expectation. They prove it is the *only* attribution that is additive over features and satisfies local accuracy, missingness and consistency at once, which is the uniqueness that makes the exponential price worth paying. Their model-agnostic estimator, KernelSHAP, recovers the $\phi_j$ from a weighted linear regression over sampled coalitions, under an added assumption of feature independence that the conditional expectation above does not itself make. The tree-specific algorithm is the model-specific case: Lundberg, Erion and Lee take exact computation for a tree ensemble from $O(TL2^{M})$ down to $O(TLD^{2})$, with $T$ trees, $L$ the largest leaf count, $M$ features and $D$ the greatest depth, by walking the trees instead of enumerating coalitions.

A note on symbols, since this vault uses $\phi$ for two things. In [[Feature Engineering]] $\phi$ is the feature map, the function applied to an instance before the hypothesis sees it. Here $\phi_j$ is the Shapley value of feature $j$, which is the convention of the game-theory literature the quantity comes from. They are unrelated and never appear in the same expression. The Shapley value as a mathematical object is not treated here, only its use as an importance; the axioms that pin it down as the unique such average, and the fact that Shapley's own 1953 statement used three of them rather than the four usually quoted, belong with the game theory this vault has not reached.

## Where it is used

[[Feature Engineering]] is the parent activity and the reason to take the measurement at all: after a fit, the importances say which of the engineered columns actually paid, and a ratio or a cross that ranks at the bottom is a candidate for removal on the next pass. [[Irrelevant Features]] is the pathology at the other end of the ranking, and $I_j$ is the finite-sample criterion that note operationalizes its conditional-independence definition with. [[Feature Generalization]] is the complementary reading on the same candidate column: importance measures what the column is worth on rows that have it, coverage measures how many rows those are, and a column scoring high on one and low on the other is the combination most worth chasing. [[Data Leakage]] sends the diagnostic use here, as set out under ablation above. [[Overfitting]] is why an importance read off training data cannot be trusted as a statement about unseen rows, which is the bias the next paragraph is about.

**The tools that implement this, and what each one actually is.** Gradient-boosted trees are the classical route, and the reason is that a tree ensemble hands you a ranking as a by-product of being fitted: every split records how much it reduced the splitting criterion, and summing that over the splits on a column, averaging over trees and normalizing to sum to one is **mean decrease in impurity**, which costs nothing extra because the numbers were computed during the fit. It is what scikit-learn 1.6 returns from `feature_importances_` on its forests and boosted ensembles.

That ranking carries a bias, scikit-learn documents it itself, and it is not small. It favours high-cardinality columns, typically the continuous ones, because a column with many distinct values offers many candidate split points and some of them reduce impurity on noise alone. And it is computed from training-set statistics, so it says nothing about whether the column helps on rows the model has not seen; the library's own worked example adds two pure-noise columns to a dataset, one continuous and one with three levels, and both come back with non-zero importance while the continuous one ranks near the top. Permutation on held-out data is the correction, and the honest use of the impurity ranking is to run both and read the disagreement, since a column that ranks high on impurity and near zero on held-out permutation is being flagged as fitted-to rather than as useful. [[Overfitting]] is the name of what is being detected there.

**XGBoost** (Chen and Guestrin, *KDD* 2016) is the gradient-boosting implementation most often reached for, and it is its own package rather than part of scikit-learn, shipping a scikit-learn compatible wrapper so that it drops into a [[Pipeline]] unchanged. That wrapper exposes `feature_importances_`, the same attribute name scikit-learn's own ensembles expose, and **the two do not return the same quantity**. Scikit-learn's is mean decrease in impurity. XGBoost's defaults to **gain**, the average loss reduction across the splits a column is used in, for any tree booster, falling back to `weight` only for the linear booster. Both are normalized to sum to one, both are read off the fit rather than off held-out data, and nothing in either call site says which measure you are looking at. The trap has a second door inside XGBoost itself: the underlying booster's own `get_score` defaults to `weight`, the raw count of splits using the column, and so does the plotting helper, so the bar chart and the attribute can rank the same fitted model differently unless the importance type is passed explicitly.

**SHAP** (Lundberg and Lee, *NeurIPS* 2017) is the `shap` package, and what it implements is the construction in the Shapley subsection above: KernelSHAP for any model, the tree algorithm where the model is a tree ensemble, and the per-instance attributions that make it a local explanation as well as a global ranking, since summing the $\phi_j$ of one row reconstructs that row's prediction.

**InterpretML** (Nori, Jenkins, Koch and Caruana, 2019) is the `interpret` package, out of Microsoft, and it does two separable things that get run together. It wraps existing black-box explainers, SHAP's kernel estimator and LIME and partial dependence and Morris sensitivity among them, so they can be compared under one interface. And it carries the Explainable Boosting Machine, which is not an explainer at all but a model: a generalized additive model with pairwise interaction terms, each feature's contribution fitted by gradient boosting on very shallow trees, cycled one feature at a time at a low learning rate so that feature order does not decide the result. Its importances need no removal experiment, because the model *is* a sum of per-feature functions and you can simply read each one off. That is a different bargain from everything else on this page, buying interpretability in the model class rather than recovering it afterwards.

**Neither of those is part of scikit-learn**, however often they sit beside it and however carefully their APIs are shaped to match, and the Explainable Boosting Machine really does subclass scikit-learn's estimator base classes rather than merely imitating `fit` and `predict`. All four are named here rather than given notes of their own, because what earns a note in this vault is the thing implemented rather than the library implementing it, and any claim about which library is in fashion is a claim with a date on it.
