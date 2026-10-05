# Machine Learning (MCC-2.3.2) — Unit II: Supervised Learning 2
**PG MCA, Ravenshaw University (CBCS) | Ref: Hastie–Tibshirani–Friedman, *ESL* 2e (Ch. 9.2, 10, 12, 15); ISLR Ch. 8–9; Chen & Guestrin (XGBoost, 2016)**

## Syllabus Topics (Unit II)
1. SVM for Binary and Multiclass Classification
2. SVM for Regression
3. Regression and Classification Trees
4. Ensemble Learning and Random Forest
5. Boosting Methods: AdaBoost, XGBoost
6. Performance of Classification Algorithms: Confusion Matrix, Precision, Recall, ROC Curve

---

## 1. Support Vector Machine (SVM) — Binary Classification

**Idea:** Find the **hyperplane** that separates two classes with the **maximum margin**. The points closest to the hyperplane are the **support vectors**; only they determine the boundary.

**Hyperplane:** `wᵀx + b = 0`. Labels `y ∈ {−1, +1}`. Prediction: `sign(wᵀx + b)`.
**Margin** = distance between the two parallel boundaries `wᵀx + b = ±1` = **2/‖w‖**.

### 1.1 Hard-Margin SVM (linearly separable data)

```
minimise   ½‖w‖²
subject to yᵢ(wᵀxᵢ + b) ≥ 1   for all i
```
Maximising the margin `2/‖w‖` ⇔ minimising `‖w‖²`. A convex quadratic program (QP) with a unique global solution.

### 1.2 Soft-Margin SVM (overlapping / noisy data)
Allow some violations using **slack variables** `ξᵢ ≥ 0`:

```
minimise   ½‖w‖² + C Σ ξᵢ
subject to yᵢ(wᵀxᵢ + b) ≥ 1 − ξᵢ ,  ξᵢ ≥ 0
```
- `ξᵢ = 0`: correct, outside margin. `0 < ξᵢ ≤ 1`: inside margin but correct. `ξᵢ > 1`: misclassified.
- **C = regularisation parameter** (trade-off):
  - Large C → narrow margin, few violations → low bias, high variance (overfit risk).
  - Small C → wide margin, more violations → high bias, low variance.
- Equivalent to minimising **hinge loss** + L2 penalty: `Σ max(0, 1 − yᵢf(xᵢ)) + λ‖w‖²`.

### 1.3 Dual Form and Support Vectors
Using Lagrange multipliers `αᵢ`:

```
maximise  Σ αᵢ − ½ ΣΣ αᵢαⱼ yᵢyⱼ (xᵢ·xⱼ)
subject to 0 ≤ αᵢ ≤ C ,  Σ αᵢyᵢ = 0
w = Σ αᵢ yᵢ xᵢ ;   f(x) = Σ αᵢ yᵢ (xᵢ·x) + b
```
- Only points with `αᵢ > 0` (the **support vectors**) matter. Data appears only through **dot products** → enables the kernel trick.

### 1.4 Kernel Trick (non-linear SVM)
Map data to a higher-dimensional space `φ(x)` where it becomes linearly separable, but compute `K(x, x') = φ(x)·φ(x')` directly without computing `φ` (replace every dot product with `K`).

| Kernel | Formula |
|---|---|
| Linear | `xᵀx'` |
| Polynomial | `(γ xᵀx' + r)ᵈ` |
| **RBF (Gaussian)** | `exp(−γ‖x − x'‖²)` |
| Sigmoid | `tanh(γ xᵀx' + r)` |

- RBF `γ`: large γ → very local, overfits; small γ → smooth, underfits.
- Hyperparameters (`C`, `γ`, kernel) are tuned by **cross-validation / grid search**. **Scale features** before SVM.

**Pros:** effective in high dimensions, memory-efficient (support vectors only), versatile via kernels, global optimum.
**Cons:** slow on very large datasets (≈ O(n²–n³)), sensitive to kernel/parameters, no direct probability output (needs Platt scaling), less interpretable.

---

## 2. SVM for Multiclass Classification

SVM is inherently binary; extended by decomposition:

| Strategy | Method | Classifiers needed |
|---|---|---|
| **One-vs-Rest (OvR / OvA)** | One SVM per class: that class vs all others. Predict the class with the **highest decision score**. | K |
| **One-vs-One (OvO)** | One SVM for every pair of classes. Predict by **majority voting**. | K(K−1)/2 |

- OvR: fewer models, but imbalanced training sets per classifier.
- OvO: more models, each trained on smaller data (often faster per model); used by `libsvm` / sklearn `SVC`.
- Other: DAG-SVM, direct multiclass formulations (Crammer–Singer).

---

## 3. SVM for Regression (Support Vector Regression, SVR)

**Idea:** Fit a function `f(x) = wᵀx + b` such that most points lie within a tube of width **ε** around it. Errors **inside the ε-tube are ignored**.

**ε-insensitive loss**

```
L_ε(y, f(x)) = max(0, |y − f(x)| − ε)
```

**Optimisation** (slack `ξᵢ, ξᵢ*` for points above/below the tube):

```
minimise   ½‖w‖² + C Σ (ξᵢ + ξᵢ*)
subject to  yᵢ − f(xᵢ) ≤ ε + ξᵢ
            f(xᵢ) − yᵢ ≤ ε + ξᵢ*
            ξᵢ, ξᵢ* ≥ 0
```
- **Support vectors** = points on or outside the tube.
- Parameters: `ε` (tube width → sparsity of support vectors), `C` (penalty for points outside tube), kernel + γ for non-linear regression.
- Larger ε → fewer support vectors, smoother fit. Robust to outliers (linear penalty beyond ε).

---

## 4. Regression and Classification Trees (CART)

**Decision tree:** A flowchart-like model that recursively **partitions the feature space** into rectangular regions using binary splits `xⱼ ≤ s`. Each **leaf** gives a prediction. Terms: root node, internal (decision) node, branch, leaf (terminal) node, depth.

### 4.1 Regression Trees
- Leaf prediction = **mean of `y`** of training points in that region.
- Choose split (variable `j`, threshold `s`) minimising the **RSS**:

```
min over (j,s) [ Σ_{xᵢ∈R₁(j,s)} (yᵢ − ĉ₁)² + Σ_{xᵢ∈R₂(j,s)} (yᵢ − ĉ₂)² ]
```

### 4.2 Classification Trees
- Leaf prediction = **majority class** in the region.
- Node impurity measures (p̂ₖ = proportion of class k in node):

| Measure | Formula |
|---|---|
| **Gini index** | `G = Σ p̂ₖ(1 − p̂ₖ) = 1 − Σ p̂ₖ²` |
| **Entropy** | `H = −Σ p̂ₖ log₂ p̂ₖ` |
| Misclassification error | `1 − max p̂ₖ` |

- **Information Gain** = impurity(parent) − weighted impurity(children). Pick the split with the best gain.
- Gini/entropy are preferred for growing (more sensitive to node purity); misclassification error for pruning.

### 4.3 Growing, Overfitting and Pruning
- **Greedy, top-down recursive binary splitting** (no backtracking) until a stopping rule (min samples, max depth).
- A fully grown tree **overfits** (low bias, high variance).
- **Cost-complexity (weakest-link) pruning:** minimise `Σ RSS(leaves) + α|T|`, where `|T|` = number of leaves; choose `α` by **cross-validation**.
- Alternatively pre-prune with `max_depth`, `min_samples_leaf`.

**Pros:** easy to interpret/visualise, handles numeric and categorical data, no scaling needed, captures non-linearity and interactions, handles missing values (surrogate splits).
**Cons:** high variance (unstable — small data change alters the tree), piecewise-constant (not smooth), greedy (not globally optimal), biased toward features with many levels.

---

## 5. Ensemble Learning and Random Forest

**Ensemble learning:** Combine predictions of **multiple models** (base/weak learners) to get a better, more stable model than any single one. "Wisdom of the crowd."

### 5.1 Ensemble Types
| Type | Idea | Reduces | Example |
|---|---|---|---|
| **Bagging** (parallel) | Train models independently on bootstrap samples; average / vote | **Variance** | Random Forest |
| **Boosting** (sequential) | Each model corrects errors of previous ones | **Bias** (and variance) | AdaBoost, XGBoost |
| **Stacking** | Meta-model learns to combine base-model outputs | Both | Stacked generalisation |
| Voting | Majority (hard) / averaged probabilities (soft) | Variance | VotingClassifier |

### 5.2 Bagging (Bootstrap Aggregating)
1. Draw `B` bootstrap samples from the training set.
2. Train a (deep, unpruned) tree on each.
3. **Regression:** average; **Classification:** majority vote.

`Var(average of B identical-variance trees with pairwise correlation ρ) = ρσ² + (1−ρ)σ²/B` → variance falls with larger `B`, but is limited by the correlation `ρ` between trees.
- **Out-of-Bag (OOB) error:** each tree leaves out ≈ 36.8 % of the data; predict those points with the trees that did not see them → a free, CV-like test error estimate.

### 5.3 Random Forest
**Bagging + random feature selection** to **decorrelate** the trees (reduces `ρ`).

**Algorithm**
1. For `b = 1..B`: draw a bootstrap sample.
2. Grow a tree; at **each split**, choose the best split from a **random subset of `m` features** (not all `p`).
   - Typical: `m ≈ √p` (classification), `m ≈ p/3` (regression).
3. Aggregate: majority vote / average.

**Key points**
- Smaller `m` → less correlated trees (lower variance) but higher bias per tree.
- Trees are fully grown (no pruning).
- **Feature importance:** mean decrease in impurity, or **permutation importance** (drop in OOB accuracy after permuting a variable).
- Hyperparameters: `n_estimators`, `max_features (m)`, `max_depth`, `min_samples_leaf`.

**Pros:** high accuracy, robust to overfitting and noise, handles high dimensions, OOB estimate, parallelisable, little tuning.
**Cons:** less interpretable than a single tree, slower/larger model, can't extrapolate beyond training range (regression).

---

## 6. Boosting Methods

**Boosting:** Build a strong learner by training **weak learners sequentially**, each focusing on what earlier ones got wrong, then combining them in a **weighted sum**:  `F(x) = Σ αₘ hₘ(x)`.

### 6.1 AdaBoost (Adaptive Boosting) — Freund & Schapire

**Weak learner:** typically a **decision stump** (1-level tree). Labels `y ∈ {−1, +1}`.

**Algorithm (AdaBoost.M1)**
1. Initialise weights `wᵢ = 1/N`.
2. For `m = 1..M`:
   1. Fit classifier `hₘ(x)` to the training data using weights `wᵢ`.
   2. Weighted error: `errₘ = Σ wᵢ·I(yᵢ ≠ hₘ(xᵢ)) / Σ wᵢ`
   3. Classifier weight: `αₘ = ½ ln((1 − errₘ)/errₘ)` (ESL uses `ln` without ½; same effect)
   4. Update weights: `wᵢ ← wᵢ · exp(−αₘ yᵢ hₘ(xᵢ))`, then normalise.
      → **Misclassified points get larger weights**, correct ones smaller.
3. Final model: `H(x) = sign( Σₘ αₘ hₘ(x) )`

**Key points**
- Better classifiers (low error) get higher `αₘ`; `errₘ > 0.5` would give negative α (worse than random).
- Equivalent to forward stagewise additive modelling with the **exponential loss** `exp(−y f(x))`.
- Sensitive to **noisy data and outliers** (their weights keep growing).
- `learning rate` shrinks each contribution to reduce overfitting.

### 6.2 Gradient Boosting (foundation of XGBoost)
Generalises boosting to **any differentiable loss**. At each stage, fit a new tree to the **negative gradient (pseudo-residuals)** of the loss at the current model, then add it with a shrinkage (learning rate) `η`:

```
F_m(x) = F_{m−1}(x) + η · h_m(x),    h_m fit to  rᵢ = −∂L(yᵢ, F)/∂F |_{F=F_{m−1}}
```
For squared loss, the pseudo-residuals are simply `yᵢ − F_{m−1}(xᵢ)` (ordinary residuals).

### 6.3 XGBoost (eXtreme Gradient Boosting) — Chen & Guestrin, 2016

An **optimised, regularised, scalable** implementation of gradient-boosted trees. Final model is an additive sum of `K` trees: `ŷᵢ = Σₖ fₖ(xᵢ)`.

**Regularised objective**

```
Obj = Σᵢ l(yᵢ, ŷᵢ)  +  Σₖ Ω(fₖ)
Ω(f) = γ·T + ½ λ Σⱼ wⱼ²          (T = number of leaves, wⱼ = leaf weights)
```
- `γ` penalises the number of leaves; `λ` is the L2 penalty on leaf weights (an optional L1 term `α` also exists).

**Second-order Taylor approximation** — with `gᵢ` (gradient) and `hᵢ` (Hessian) of the loss:

```
Obj⁽ᵗ⁾ ≈ Σᵢ [ gᵢ f_t(xᵢ) + ½ hᵢ f_t(xᵢ)² ] + Ω(f_t)
Optimal leaf weight:  wⱼ* = − Gⱼ / (Hⱼ + λ)       (Gⱼ = Σgᵢ, Hⱼ = Σhᵢ in leaf j)
Split gain = ½ [ G_L²/(H_L+λ) + G_R²/(H_R+λ) − (G_L+G_R)²/(H_L+H_R+λ) ] − γ
```
A split is made only if **Gain > 0** (γ acts as built-in pruning).

**Why XGBoost is better than plain gradient boosting**
| Feature | Benefit |
|---|---|
| Regularisation (γ, λ, α) in objective | Less overfitting |
| Uses 2nd-order (Hessian) info | More accurate steps, faster convergence |
| Shrinkage (learning rate η), column & row subsampling | Reduces overfitting |
| **Sparsity-aware** split finding, learns default direction for missing values | Handles missing/sparse data |
| Weighted quantile sketch, histogram approx. splits | Scales to big data |
| Parallel/cache-aware block structure, out-of-core computing | Speed |
| Built-in cross-validation, early stopping | Easy tuning |

**Key hyperparameters:** `n_estimators`, `learning_rate (η)`, `max_depth`, `min_child_weight`, `subsample`, `colsample_bytree`, `gamma`, `lambda`, `alpha`.

### 6.4 Bagging vs Boosting vs AdaBoost vs XGBoost
| | Bagging / RF | AdaBoost | XGBoost |
|---|---|---|---|
| Training | Parallel | Sequential | Sequential (parallel tree construction) |
| Focus | Reduce variance | Reweight misclassified samples | Fit gradient (2nd-order) with regularisation |
| Base learner | Deep trees | Stumps | Shallow trees |
| Weighting | Equal vote | Weighted by α | Shrunk by η |
| Outlier sensitivity | Low | High | Moderate |

---

## 7. Performance of Classification Algorithms

### 7.1 Confusion Matrix
A table comparing **actual** vs **predicted** classes (binary case, positive class = class of interest):

|  | **Predicted Positive** | **Predicted Negative** |
|---|---|---|
| **Actual Positive** | **TP** (True Positive) | **FN** (False Negative — Type II error) |
| **Actual Negative** | **FP** (False Positive — Type I error) | **TN** (True Negative) |

For `K` classes it is a `K × K` matrix; diagonal = correct predictions.

### 7.2 Metrics
| Metric | Formula | Meaning |
|---|---|---|
| **Accuracy** | `(TP + TN) / (TP + TN + FP + FN)` | Overall fraction correct (misleading for imbalanced data) |
| **Precision** (Positive Predictive Value) | `TP / (TP + FP)` | Of predicted positives, how many are truly positive |
| **Recall** (Sensitivity, TPR) | `TP / (TP + FN)` | Of actual positives, how many were found |
| **Specificity** (TNR) | `TN / (TN + FP)` | Of actual negatives, how many correctly identified |
| **False Positive Rate (FPR)** | `FP / (FP + TN) = 1 − Specificity` | Fraction of negatives wrongly flagged |
| **F1-score** | `2·P·R / (P + R)` | Harmonic mean of precision and recall |
| Error rate | `1 − Accuracy` | |

- **Precision–Recall trade-off:** raising the decision threshold → precision ↑, recall ↓ (and vice versa).
- **Use precision** when false positives are costly (spam filter, recommendations). **Use recall** when false negatives are costly (cancer screening, fraud detection).
- **Multiclass averaging:** *macro* (simple average per class), *micro* (pool TP/FP/FN globally), *weighted* (average weighted by class support).

**Worked example:** TP = 40, FN = 10, FP = 5, TN = 45 (N = 100)
- Accuracy = 85/100 = **0.85**
- Precision = 40/45 = **0.889**
- Recall = 40/50 = **0.80**
- Specificity = 45/50 = **0.90**
- F1 = 2(0.889)(0.80)/(0.889+0.80) ≈ **0.842**

### 7.3 ROC Curve and AUC
**ROC (Receiver Operating Characteristic) curve:** plot of **TPR (Recall)** on the y-axis against **FPR** on the x-axis, obtained by **varying the classification threshold** from 0 to 1.

- Each point = one threshold. **(0,0)**: predict all negative; **(1,1)**: predict all positive; **(0,1)**: perfect classifier (top-left corner).
- **Diagonal line** (TPR = FPR) = random guessing.
- Curve closer to top-left = better classifier.

**AUC (Area Under the ROC Curve):**
| AUC | Interpretation |
|---|---|
| 1.0 | Perfect |
| 0.9 – 1.0 | Excellent |
| 0.8 – 0.9 | Good |
| 0.5 | No better than random |
| < 0.5 | Worse than random (labels likely inverted) |

- AUC = probability that a randomly chosen positive is ranked higher than a randomly chosen negative.
- Threshold-independent and insensitive to class prior; used to compare models and choose an operating threshold.
- For **highly imbalanced data**, the **Precision–Recall curve** is more informative than ROC.

---

## Quick Revision Sheet

| Topic | One-line takeaway |
|---|---|
| SVM | Max-margin hyperplane; margin = 2/‖w‖; support vectors define it |
| Soft margin / C | Slack ξ allowed; C trades margin width vs errors |
| Kernel trick | Replace dot product with K(x,x'); RBF most common |
| Multiclass SVM | OvR (K models, max score) / OvO (K(K−1)/2 models, vote) |
| SVR | ε-insensitive tube; errors inside ε ignored |
| Regression tree | Split to minimise RSS; leaf = mean |
| Classification tree | Split by Gini/entropy; leaf = majority class |
| Pruning | Cost-complexity `RSS + α\|T\|`, α via CV |
| Bagging | Bootstrap + average → ↓ variance |
| Random Forest | Bagging + random m features per split → decorrelated trees; OOB error |
| AdaBoost | Reweight misclassified points; α = ½ln((1−err)/err); weighted vote |
| Gradient boosting | Fit trees to negative gradient of loss |
| XGBoost | Regularised (γ, λ) + 2nd-order Taylor + engineering optimisations |
| Confusion matrix | TP, FP, FN, TN |
| Precision / Recall | TP/(TP+FP) ; TP/(TP+FN) |
| ROC / AUC | TPR vs FPR over thresholds; AUC=1 perfect, 0.5 random |

## Likely Exam Questions
1. Explain SVM with the concepts of margin, support vectors and slack variables. What is the role of C?
2. What is the kernel trick? Name common kernels.
3. How is SVM extended to multiclass classification? Compare OvR and OvO.
4. Explain Support Vector Regression and the ε-insensitive loss.
5. Explain how regression and classification trees are built; Gini vs entropy; how is pruning done?
6. Differentiate bagging and boosting. Explain how Random Forest reduces variance.
7. Explain the AdaBoost algorithm step by step.
8. How does XGBoost differ from standard gradient boosting? Write its objective function.
9. Define confusion matrix, precision, recall, F1; compute them from given values.
10. What is a ROC curve? Explain AUC and how the curve is drawn.

---
*Aligned to the prescribed syllabus: ESL (Hastie, Tibshirani, Friedman) — Ch. 9.2 (trees), Ch. 10 (boosting/AdaBoost), Ch. 12 (SVM), Ch. 15 (Random Forests); XGBoost from Chen & Guestrin (KDD 2016).*
