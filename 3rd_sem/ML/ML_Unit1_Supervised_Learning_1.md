# Machine Learning (MCC-2.3.2) — Unit I: Supervised Learning 1
**PG MCA, Ravenshaw University (CBCS) | Ref: Hastie–Tibshirani–Friedman, *ESL* 2e (Ch. 2, 3, 4, 7, 8.5); ISLR; Mitchell**

## Syllabus Topics (Unit I)
1. Overview of Supervised Learning
2. Classification and Regression problems
3. K-Nearest Neighbour (KNN) Classifier
4. Naïve Bayes Classifier
5. Multiple Linear Regression
6. Shrinkage Methods: Ridge, Lasso, Elastic Net
7. Logistic Regression
8. Linear Discriminant Analysis (LDA)
9. Feature Subset Selection
10. Loss Function
11. Test and Training Error
12. Bias, Variance, Model Complexity
13. Bias–Variance Trade-off
14. Cross-Validation
15. Bootstrap Methods

---

## 1. Overview of Supervised Learning

**Definition:** Learning a mapping **f: X → Y** from a *labelled* dataset `{(x₁,y₁), …, (xₙ,yₙ)}`. A "supervisor" (the labels) tells the model the correct output; the goal is to predict `y` for **unseen** `x`.

**Terminology**
| Term | Meaning |
|---|---|
| Input / predictor / feature / independent variable (X) | What we measure |
| Output / response / target / dependent variable (Y) | What we predict |
| Training set | Data used to fit the model |
| Test set | Unseen data used to measure generalisation |
| Hypothesis / model | The learned function f̂(x) |

**Workflow:** Collect data → preprocess → split (train/validation/test) → choose model → train (minimise loss) → evaluate → tune → deploy.

**Two approaches to prediction (ESL §2.3):**
- **Linear model / least squares:** assumes a rigid structure → low variance, possibly high bias.
- **Nearest neighbours:** very flexible, local → low bias, high variance.

---

## 2. Classification vs Regression

| | Classification | Regression |
|---|---|---|
| Output Y | Discrete / categorical (class label) | Continuous (real number) |
| Goal | Assign input to a class | Predict a quantity |
| Examples | Spam / not spam, disease yes/no | House price, temperature |
| Typical loss | 0-1 loss, cross-entropy | Squared error |
| Algorithms | KNN, Naïve Bayes, Logistic Reg., LDA | Linear, Ridge, Lasso |

> KNN works for both. Logistic Regression is a **classification** method despite its name.

---

## 3. K-Nearest Neighbour (KNN) Classifier

**Idea:** *"Tell me who your neighbours are and I'll tell you who you are."* A **lazy, non-parametric, instance-based** learner — no explicit training; it stores the data and computes at prediction time.

**Algorithm**
1. Choose `k` and a distance metric.
2. For a new point `x`, compute distance to all training points.
3. Pick the `k` nearest points.
4. **Classification:** majority vote. **Regression:** average of their `y`.

**Distance metrics**
- Euclidean: `d(x,x') = √Σ(xᵢ − x'ᵢ)²`
- Manhattan: `Σ|xᵢ − x'ᵢ|`
- Minkowski: `(Σ|xᵢ − x'ᵢ|ᵖ)^(1/p)`

**Choosing k**
- Small `k` (e.g., 1): low bias, **high variance**, overfits, jagged boundary.
- Large `k`: high bias, low variance, smooth boundary, may underfit.
- Choose `k` by **cross-validation**; use odd `k` for binary problems to avoid ties.
- Effective number of parameters ≈ `N/k`.

**Pros:** simple, no training, adapts to any boundary shape.
**Cons:** slow prediction (O(n·d)), needs **feature scaling**, suffers from the **curse of dimensionality**, sensitive to irrelevant features and noise.

**Example:** `k=3`, neighbours' classes = {A, A, B} → predict **A**.

---

## 4. Naïve Bayes Classifier

**Idea:** A **probabilistic generative** classifier based on **Bayes' theorem** with the *"naïve"* assumption that features are **conditionally independent given the class**.

**Bayes' theorem**

```
P(C | x) = P(x | C) · P(C) / P(x)
posterior = likelihood × prior / evidence
```

**Naïve assumption**

```
P(x₁,…,x_d | C) = Π P(xⱼ | C)
```

**Decision rule (MAP):**

```
ŷ = argmax_C  P(C) · Π_j P(xⱼ | C)
```
(`P(x)` is the same for all classes, so it is dropped. In practice use log-probabilities to avoid underflow.)

**Variants**
- **Gaussian NB:** continuous features, `P(xⱼ|C)` ~ Normal(μ, σ²).
- **Multinomial NB:** word counts (text classification).
- **Bernoulli NB:** binary features.

**Laplace smoothing:** add 1 to counts so an unseen feature value does not give probability 0.
`P(xⱼ|C) = (count + 1) / (N_C + |V|)`

**Pros:** fast, works with small data and high dimensions, good for text/spam.
**Cons:** independence assumption rarely true; probability estimates can be poorly calibrated; zero-frequency problem.

---

## 5. Multiple Linear Regression (MLR)

**Model:** Predicts continuous `y` from `p` predictors.

```
y = β₀ + β₁x₁ + β₂x₂ + … + β_p x_p + ε        (matrix form: y = Xβ + ε)
```

**Least Squares estimation:** minimise the Residual Sum of Squares

```
RSS(β) = Σ (yᵢ − β₀ − Σ βⱼxᵢⱼ)² = (y − Xβ)ᵀ(y − Xβ)
β̂ = (XᵀX)⁻¹ Xᵀ y          (Normal Equation)
```

**Assumptions:** linearity, independence of errors, constant variance (homoscedasticity), normally distributed errors, **no multicollinearity**.

**Evaluation:** `R² = 1 − RSS/TSS`, Adjusted R², RMSE.

**Gauss–Markov theorem:** the least-squares estimate has the *smallest variance among all linear unbiased estimators* (BLUE).

**Problems of OLS:** high variance when predictors are correlated or `p` is large; `XᵀX` non-invertible if `p > n` or collinear; poor interpretability with many predictors → motivates **shrinkage** and **subset selection**.

---

## 6. Shrinkage Methods (Regularisation)

**Idea:** Add a **penalty on coefficient size** to the loss. This accepts a *small increase in bias* for a *large reduction in variance* → better prediction. Standardise predictors first; the intercept is not penalised.

### 6.1 Ridge Regression (L2 penalty)

```
β̂_ridge = argmin { Σ(yᵢ − β₀ − Σβⱼxᵢⱼ)² + λ Σ βⱼ² }
closed form: β̂ = (XᵀX + λI)⁻¹ Xᵀ y
```
- `λ ≥ 0` is the tuning parameter (chosen by CV). `λ=0` → OLS; `λ→∞` → coefficients → 0.
- **Shrinks** coefficients toward zero but **never exactly zero** → no feature selection.
- Handles **multicollinearity** well; always invertible (`XᵀX + λI`).

### 6.2 Lasso Regression (L1 penalty)

```
β̂_lasso = argmin { Σ(yᵢ − β₀ − Σβⱼxᵢⱼ)² + λ Σ |βⱼ| }
```
- **LASSO** = Least Absolute Shrinkage and Selection Operator.
- L1 penalty forces some coefficients to become **exactly zero** → **sparse model + automatic feature selection**.
- No closed form (solved by coordinate descent / LARS).
- Weakness: with correlated predictors it tends to pick one arbitrarily; at most `n` non-zero variables when `p > n`.

**Geometry:** the L2 constraint region is a *circle/sphere* (no corners); the L1 region is a *diamond* with corners on the axes, where the RSS ellipse is likely to touch → zero coefficients.

### 6.3 Elastic Net (L1 + L2)

```
β̂_EN = argmin { RSS + λ [ α Σ|βⱼ| + (1−α) Σ βⱼ² ] }
```
- Combines Ridge and Lasso (`α=1` → Lasso, `α=0` → Ridge).
- Selects variables **and** keeps/shrinks groups of **correlated** predictors together (grouping effect).
- Good when `p ≫ n` or features are highly correlated.

| | Ridge | Lasso | Elastic Net |
|---|---|---|---|
| Penalty | L2 (Σβ²) | L1 (Σ\|β\|) | L1 + L2 |
| Sparsity | No | Yes | Yes |
| Correlated features | Handles well | Picks one | Handles well (grouped) |
| Closed form | Yes | No | No |

---

## 7. Logistic Regression

**Purpose:** **Classification** (binary; extended to multiclass by softmax). Models the **probability** of class 1.

**Sigmoid (logistic) function**

```
p(x) = P(Y=1|x) = 1 / (1 + e^−(β₀ + βᵀx))
```
**Logit (log-odds) is linear:**

```
log( p / (1 − p) ) = β₀ + β₁x₁ + … + β_p x_p
```
**Decision rule:** predict class 1 if `p ≥ 0.5` (threshold adjustable) → linear decision boundary `β₀ + βᵀx = 0`.

**Parameter estimation:** **Maximum Likelihood Estimation** (not least squares). Maximise

```
ℓ(β) = Σ [ yᵢ log pᵢ + (1 − yᵢ) log(1 − pᵢ) ]
```
(equivalently minimise **cross-entropy / log loss**). No closed form → solved iteratively using **Newton–Raphson / IRLS** or gradient descent.

**Interpretation:** `e^βⱼ` = **odds ratio** for a one-unit increase in `xⱼ`.
**Regularisation:** L1/L2 penalties can be added to prevent overfitting.

---

## 8. Linear Discriminant Analysis (LDA)

**Idea:** A **generative** classifier. Model each class density as a **multivariate Gaussian with a common covariance matrix Σ**, then use Bayes' rule. Also used for **dimensionality reduction** (Fisher's discriminant: find projection maximising between-class separation relative to within-class scatter).

**Assumptions:** `X | class k ~ N(μₖ, Σ)` (same Σ for all classes), priors `πₖ`.

**Linear discriminant function**

```
δₖ(x) = xᵀ Σ⁻¹ μₖ − ½ μₖᵀ Σ⁻¹ μₖ + log πₖ
Classify to  k̂ = argmax_k δₖ(x)
```
- Estimates: `π̂ₖ = Nₖ/N`, `μ̂ₖ` = class mean, `Σ̂` = pooled within-class covariance.
- Decision boundaries between classes are **linear** (hyperplanes).
- **QDA:** allows a separate Σₖ per class → quadratic boundaries (more flexible, more parameters).

**LDA vs Logistic Regression:** both give linear boundaries. LDA assumes Gaussian features (more efficient if true); Logistic Regression makes fewer assumptions (more robust).

---

## 9. Feature Subset Selection

**Why:** reduce overfitting, improve accuracy, interpretability, and training cost. OLS often has low bias but high variance with many predictors.

| Method | How it works |
|---|---|
| **Best subset selection** | Fit all 2ᵖ subsets; choose best per size `k`, then pick `k` by CV/AIC/BIC. Exhaustive; feasible only for small `p` (≲ 30–40). |
| **Forward stepwise** | Start with intercept only; add the variable giving the best improvement each step. |
| **Backward stepwise** | Start with all variables; remove the least significant each step (needs `n > p`). |
| **Hybrid / stepwise both ways** | Add and drop variables at each step. |
| **Shrinkage (Lasso)** | Embedded selection through L1 penalty. |

**Selection criteria:** Adjusted R², Mallows' Cp, **AIC** (`−2 log L + 2d`), **BIC** (`−2 log L + d·log n`), cross-validation error.

**Categories (general ML view):** *Filter* (statistical scores, e.g., correlation), *Wrapper* (search using model performance, e.g., stepwise), *Embedded* (selection during training, e.g., Lasso).

---

## 10. Loss Function

**Definition:** `L(Y, f̂(X))` measures the **penalty for the error** between the true value `Y` and the prediction `f̂(X)`. Training minimises the average loss over the data (**empirical risk**).

| Task | Loss | Formula |
|---|---|---|
| Regression | **Squared error (L2)** | `(y − f̂(x))²` |
| Regression | **Absolute error (L1)** | `\|y − f̂(x)\|` |
| Classification | **0-1 loss** | `I(y ≠ Ĝ(x))` |
| Classification | **Cross-entropy / log loss** | `−Σ yₖ log p̂ₖ(x)` |

- **Expected Prediction Error (EPE)** under squared loss is minimised by the **regression function** `f(x) = E[Y|X=x]` (conditional mean).
- Under 0-1 loss, the optimum is the **Bayes classifier** — choose the class with the highest posterior probability. Its error is the **Bayes error rate** (irreducible minimum).
- **Cost function** = loss averaged over the training set (+ penalty term when regularised).

---

## 11. Test Error and Training Error

| | Training error | Test error (Generalisation error) |
|---|---|---|
| Definition | Average loss on the **training** data | Expected loss on **new, unseen** data |
| Formula | `err̄ = (1/N) Σ L(yᵢ, f̂(xᵢ))` | `Err = E[L(Y, f̂(X))]` |
| Trend with complexity | **Always decreases** | **U-shaped** (decreases, then increases) |
| Usable for model selection? | No (optimistically biased) | Yes (goal to minimise) |

- **Overfitting:** very low training error but high test error (model fits noise).
- **Underfitting:** both errors high (model too simple).
- Test error is estimated using a **hold-out set, cross-validation, or bootstrap**.
- Data split: **Train** (fit) / **Validation** (tune/select) / **Test** (final assessment, used once).

---

## 12. Bias, Variance and Model Complexity

For a prediction at point `x₀`, with `Y = f(X) + ε`, `Var(ε) = σ²`:

```
EPE(x₀) = σ²  +  [Bias(f̂(x₀))]²  +  Var(f̂(x₀))
          ───     ─────────────      ────────────
     irreducible    error from        error from
        noise     wrong assumptions  sensitivity to data
```

- **Bias:** `E[f̂(x₀)] − f(x₀)` — error from *oversimplified* assumptions (a straight line fitted to a curve).
- **Variance:** how much `f̂` changes if trained on a different training sample — error from *oversensitivity* to noise.
- **Irreducible error (σ²):** noise in the data; cannot be removed by any model.
- **Model complexity:** flexibility of the model (e.g., polynomial degree, `1/k` in KNN, number of parameters, `1/λ` in regularisation).

| Complexity | Bias | Variance | Result |
|---|---|---|---|
| Low (simple) | High | Low | Underfitting |
| High (flexible) | Low | High | Overfitting |

**Examples:** Linear regression → high bias, low variance. 1-NN → low bias, high variance. Large `k` in KNN → high bias, low variance.

---

## 13. Bias–Variance Trade-off

**Concept:** As model complexity increases, **bias falls but variance rises**. Total test error = Bias² + Variance + σ², so it is minimised at an **intermediate complexity** (the "sweet spot").

```
Error
 │ \                                  /  Test error (U-shape)
 │  \         Variance ↑             /
 │   \      ______                  /
 │    \____/      \______         _/
 │     Bias² ↓            ‾‾‾‾‾‾‾‾
 │ ─ ─ ─ ─ ─ ─ ─ ─ Training error ─ ─ ↓ (keeps falling)
 └─────────────────────────────────────→ Model complexity
   underfit        optimum        overfit
```

**How to handle it**
- Select complexity using **cross-validation**.
- Use **regularisation** (Ridge/Lasso: ↑λ → ↑bias, ↓variance).
- More training data → lowers variance.
- Feature selection / dimensionality reduction.
- **Ensembles** (bagging reduces variance; boosting reduces bias — Unit II).

---

## 14. Cross-Validation (CV)

**Purpose:** Estimate **test error** reliably and **choose tuning parameters** (k, λ, model complexity) when data is limited.

### 14.1 Validation Set (Hold-out) Approach
Randomly split data into train and validation sets (e.g., 70/30). *Simple but results vary with the split and waste data.*

### 14.2 K-Fold Cross-Validation
1. Split data into `K` roughly equal folds (commonly **K = 5 or 10**).
2. For `i = 1..K`: train on `K−1` folds, test on fold `i`.
3. Average the K errors:

```
CV(f̂) = (1/N) Σ L(yᵢ, f̂^(−κ(i))(xᵢ))
```
- Use **stratified K-fold** for classification to keep class proportions.
- Choose tuning parameter value that minimises CV error (or the **one-standard-error rule**: simplest model within 1 SE of the best).

### 14.3 Leave-One-Out CV (LOOCV)
`K = N`. Train on `N−1` points, test on the one left out. Low bias, **high variance**, computationally expensive (for linear models there's a shortcut formula using the hat matrix).

| K | Bias of error estimate | Variance | Cost |
|---|---|---|---|
| Small (e.g., 2–5) | Higher (less training data) | Lower | Low |
| K = 10 | Moderate | Moderate | Moderate (good compromise) |
| K = N (LOOCV) | Lowest | Highest | High |

**Important pitfall (ESL 7.10.2):** perform feature selection / preprocessing **inside** each CV fold; doing it on the full data first gives over-optimistic results (data leakage).

---

## 15. Bootstrap Methods

**Definition:** A **resampling** technique — draw `B` samples of size `N` **with replacement** from the original data to estimate the **sampling distribution** of a statistic (standard error, bias, confidence intervals) without distributional assumptions.

**Procedure**
1. From the dataset `Z` of size `N`, draw a bootstrap sample `Z*ᵇ` (N draws with replacement).
2. Compute the statistic `S(Z*ᵇ)`.
3. Repeat `B` times (e.g., 200–1000).
4. Use the `B` values to estimate:

```
SE_boot = √[ 1/(B−1) Σ ( S(Z*ᵇ) − S̄* )² ]
```
- Each bootstrap sample contains ≈ **63.2 %** of unique original observations (`1 − (1 − 1/N)ᴺ ≈ 1 − e⁻¹`); the remaining ≈ **36.8 %** are **out-of-bag (OOB)**.

**Uses**
- Standard error and **confidence intervals** of estimators (including coefficients).
- **Estimating prediction error:** train on a bootstrap sample, test on points not in it (**leave-one-out bootstrap / OOB error**); the **.632 estimator** corrects its bias:
  `Err(.632) = 0.368 · err̄ + 0.632 · Err_boot`
- Foundation of **Bagging** and **Random Forests** (Unit II).

| | Cross-Validation | Bootstrap |
|---|---|---|
| Sampling | Without replacement (partitions) | With replacement |
| Main use | Test error / model selection | Uncertainty (SE, CI) of estimates; error estimation |
| Training-set size | (K−1)/K of data | ~63.2 % unique points |

---

## Quick Revision Sheet

| Topic | One-line takeaway |
|---|---|
| Supervised learning | Learn f(X)→Y from labelled data |
| Classification / Regression | Discrete vs continuous output |
| KNN | Majority vote of k nearest points; lazy; scale features |
| Naïve Bayes | Bayes theorem + conditional independence; argmax P(C)ΠP(xⱼ\|C) |
| MLR | β̂ = (XᵀX)⁻¹Xᵀy; minimises RSS |
| Ridge | L2; shrinks, never zero; fixes multicollinearity |
| Lasso | L1; exact zeros; feature selection |
| Elastic Net | L1 + L2; grouped selection |
| Logistic Regression | Sigmoid of linear score; MLE; linear boundary |
| LDA | Gaussian classes, shared Σ; linear discriminant |
| Subset selection | Best subset / forward / backward; AIC, BIC, CV |
| Loss function | Penalty for prediction error (squared, 0-1, cross-entropy) |
| Train vs test error | Train ↓ always; test is U-shaped |
| Bias–Variance | Err = σ² + Bias² + Variance |
| Trade-off | Pick intermediate complexity via CV/regularisation |
| Cross-validation | K-fold (K=5/10), LOOCV; estimate test error |
| Bootstrap | Resample with replacement; SE, CI, OOB error |

## Likely Exam Questions
1. Differentiate classification and regression with examples.
2. Explain the KNN algorithm; how does `k` affect bias and variance?
3. Derive/explain the Naïve Bayes classifier; what is Laplace smoothing?
4. Compare Ridge, Lasso and Elastic Net.
5. Explain logistic regression and the sigmoid function; how are parameters estimated?
6. Explain LDA and its assumptions; compare with logistic regression.
7. Explain forward, backward and best subset selection.
8. Explain the bias–variance decomposition and trade-off with a diagram.
9. Explain K-fold CV and LOOCV; compare them.
10. Describe the bootstrap method and its uses.

---
*Sources aligned to the prescribed syllabus: ESL (Hastie, Tibshirani, Friedman) — Ch. 2 (overview, KNN, bias–variance), 3.1–3.4 (linear regression, subset selection, ridge, lasso), 4.3–4.4 (LDA, logistic regression), 7 (loss, error, bias–variance, CV, bootstrap), 8.5; ISLR Ch. 2, 5, 6 for supporting explanations.*
