# Machine Learning (MCC-2.3.2) — Unit III: Unsupervised Learning
**PG MCA, Ravenshaw University (CBCS) | Ref: Hastie–Tibshirani–Friedman, *ESL* 2e (Ch. 13.2, 14.3, 14.5–14.7); ISLR Ch. 12; Bishop PRML Ch. 9, 12; Alpaydin Ch. 7, 9**

## Syllabus Topics
1. Clustering: K-means, Spectral Clustering
2. Feature Extraction: PCA, Kernel PCA, Independent Component Analysis (printed as "IDA" in the syllabus — it is **ICA**)
3. Non-negative Matrix Factorization (NMF) and Collaborative Filtering
4. Mixture of Gaussians and its application to Clustering
5. Anomaly Detection
6. Novelty Detection
7. Learning Vector Quantization (LVQ)

---

## 0. Unsupervised Learning — Overview
**Definition:** Learning patterns/structure from **unlabelled** data `{x₁,…,x_N}` (no response `y`). 

| Task | Goal | Methods in this unit |
|---|---|---|
| **Clustering** | Group similar observations | K-means, spectral, GMM |
| **Dimensionality reduction / feature extraction** | Find a compact representation | PCA, kernel PCA, ICA, NMF |
| **Density estimation** | Model `p(x)` | GMM |
| **Anomaly / novelty detection** | Find unusual points | GMM, one-class methods, etc. |
| **Matrix completion / recommendation** | Fill unknown entries | NMF, collaborative filtering |

Difficulty: no ground truth → evaluation is subjective/indirect (internal metrics, domain knowledge).

---

## 1. Clustering

**Definition:** Partition data into groups (**clusters**) so that points in the same cluster are **similar** (high intra-cluster similarity) and points in different clusters are **dissimilar**.

**Types:** Partitional (K-means), Hierarchical (agglomerative/divisive), Density-based (DBSCAN), Model-based (GMM), Graph-based (spectral). **Hard** assignment (point belongs to one cluster) vs **soft** (probabilities).

### 1.1 K-Means Clustering

**Idea:** Partition `N` points into **K** clusters, each represented by its **centroid (mean)**; minimise the **within-cluster sum of squares (WCSS / inertia)**:

```
J = Σ_{k=1}^{K} Σ_{xᵢ ∈ C_k} ‖xᵢ − μ_k‖²
```

**Algorithm (Lloyd's)**
1. Choose `K`; initialise centroids `μ₁..μ_K` (random points / **K-means++**).
2. **Assignment step:** assign each `xᵢ` to the nearest centroid: `cᵢ = argmin_k ‖xᵢ − μ_k‖²`.
3. **Update step:** `μ_k = mean of all points assigned to cluster k`.
4. Repeat 2–3 until assignments stop changing (or `J` change < tolerance).

**Properties**
- Each step **never increases `J`** → guaranteed convergence, but only to a **local minimum** → run with several random initialisations (`n_init`) and keep the best.
- Time ≈ **O(N·K·d·iterations)**; scalable.
- It is a special case of EM for a GMM with equal, spherical covariances and hard assignments.

**Choosing K**
- **Elbow method:** plot `J` vs `K`; pick the "elbow".
- **Silhouette score:** `s = (b − a)/max(a, b)` (a = mean intra-cluster distance, b = mean distance to nearest other cluster); in [−1, 1], higher is better.
- Gap statistic.

**Limitations:** must pre-specify K; assumes **spherical, similar-sized, convex** clusters; sensitive to **outliers**, initialisation and **feature scale** (standardise first); only for numeric data (K-medoids/K-modes for others).

**Worked mini-example:** points {1, 2, 10, 11}, K=2, initial centroids 1 and 10 → clusters {1,2} and {10,11} → centroids 1.5 and 10.5 → no change → converged.

### 1.2 Spectral Clustering

**Idea:** Use the **eigenvectors of a graph Laplacian** built from a similarity graph to embed data in a low-dimensional space where clusters become easy to separate (then run K-means). Works for **non-convex** clusters (e.g., concentric rings, moons) where K-means fails.

**Algorithm (normalised spectral clustering)**
1. Build a **similarity (affinity) matrix** `W`: e.g., Gaussian kernel `Wᵢⱼ = exp(−‖xᵢ−xⱼ‖²/2σ²)`, or a k-nearest-neighbour / ε-neighbourhood graph.
2. Degree matrix `D` (diagonal, `Dᵢᵢ = Σⱼ Wᵢⱼ`).
3. **Graph Laplacian:** unnormalised `L = D − W`; normalised `L_sym = I − D^(−1/2) W D^(−1/2)`.
4. Compute the **K smallest eigenvectors** of `L` → stack as columns of `U ∈ R^{N×K}` (normalise rows for `L_sym`).
5. Treat each row of `U` as a new point in `R^K`; run **K-means**.
6. Assign original points the cluster of their row.

**Why it works:** it approximately minimises a **graph cut** criterion (Ratio Cut / Normalised Cut) — cutting few edges between clusters while keeping clusters balanced. Number of zero eigenvalues of `L` = number of connected components; the **eigengap** suggests K.

**Pros:** handles arbitrary shapes, works from just a similarity matrix. **Cons:** O(N³) eigendecomposition (costly for large N), sensitive to the choice of `σ` / graph construction, K still needed.

---

## 2. Feature Extraction (Dimensionality Reduction)

**Why:** curse of dimensionality, noise removal, visualisation, compression, faster learning. 
**Feature selection** keeps a subset of original features; **feature extraction** creates **new features** (combinations of originals).

### 2.1 Principal Component Analysis (PCA)

**Idea:** Find orthogonal directions (**principal components**) along which the data has **maximum variance**; project data onto the top `q` of them. An **unsupervised, linear** technique.

**Equivalent views:** (a) maximise variance of the projection; (b) minimise the mean squared **reconstruction error**.

**Algorithm**
1. **Centre** the data (subtract mean); **standardise** if features have different scales.
2. Compute covariance matrix `S = (1/N) XᵀX` (X centred).
3. Eigendecomposition: `S vⱼ = λⱼ vⱼ`, with `λ₁ ≥ λ₂ ≥ … ≥ λ_p`.
   (Equivalently, **SVD** of `X = U D Vᵀ`; columns of `V` = principal directions, `λⱼ = dⱼ²/N`.)
4. Select the top `q` eigenvectors `V_q = [v₁..v_q]`.
5. **Project:** `Z = X V_q` (the scores).  **Reconstruct:** `X̂ = Z V_qᵀ`.

**Key facts**
- `λⱼ` = variance along PC j. **Proportion of variance explained** = `λⱼ / Σλ`. Choose `q` so cumulative variance ≥ 90–95 %, or use a **scree plot** elbow.
- PCs are **uncorrelated** and orthogonal; first PC captures the most variance.
- Limitations: only **linear** structure; sensitive to scaling and outliers; PCs are hard to interpret; variance ≠ relevance for prediction.

### 2.2 Kernel PCA

**Idea:** **Non-linear PCA** — apply PCA in a high-dimensional **feature space** `φ(x)` using the **kernel trick**, without computing `φ` explicitly.

**Algorithm**
1. Choose a kernel `K(x, x')` (RBF `exp(−γ‖x−x'‖²)`, polynomial, …). Compute the `N×N` kernel (Gram) matrix `Kᵢⱼ = K(xᵢ, xⱼ)`.
2. **Centre** the kernel matrix: `K̃ = K − 1_N K − K 1_N + 1_N K 1_N` (`1_N` = N×N matrix of 1/N).
3. Solve the eigenproblem `K̃ aₖ = N λₖ aₖ`; normalise so that `λₖ aₖᵀaₖ = 1`.
4. **Projection** of a point `x` on the k-th component: `zₖ(x) = Σᵢ aₖᵢ K̃(xᵢ, x)`.

**Notes:** can give up to `N` components; captures non-linear manifolds (e.g., concentric circles become separable); no direct inverse mapping (pre-image problem); O(N³) cost; kernel parameters need tuning. Linear kernel ⇒ ordinary PCA.

### 2.3 Independent Component Analysis (ICA)

**Idea:** Separate a multivariate signal into **statistically independent, non-Gaussian** source components. Classic application: **Blind Source Separation / "cocktail-party problem"** (recover individual speakers from mixed microphone recordings).

**Model:** observed `x = A s`, where `s` = independent sources, `A` = unknown **mixing matrix**. Goal: find **unmixing matrix** `W ≈ A⁻¹` so `ŝ = W x` has independent components.

**Preprocessing:** centre and **whiten** (decorrelate + unit variance, typically via PCA) → reduces the problem to finding an **orthogonal rotation**.

**Principle:** By the **central limit theorem**, a mixture of independent sources is **more Gaussian** than the sources. So find directions that **maximise non-Gaussianity**, measured by **kurtosis** or **negentropy** (e.g., FastICA fixed-point algorithm), or equivalently **minimise mutual information** / maximise likelihood (Infomax).

**Requirements:** sources must be (i) statistically independent, (ii) **non-Gaussian** (at most one Gaussian source), (iii) number of mixtures ≥ number of sources.

**Ambiguities:** order of components and their scale/sign cannot be determined.

| | PCA | ICA |
|---|---|---|
| Goal | Maximise variance | Maximise independence / non-Gaussianity |
| Components | Uncorrelated, orthogonal | Statistically independent (stronger than uncorrelated), not necessarily orthogonal |
| Statistics used | 2nd-order (covariance) | Higher-order (kurtosis, negentropy) |
| Ordering | By variance | No natural order |
| Typical use | Compression, noise reduction | Signal separation (audio, EEG/fMRI) |

---

## 3. Non-negative Matrix Factorization (NMF) and Collaborative Filtering

### 3.1 NMF
**Idea:** Approximate a **non-negative** data matrix `V (n×m)` as a product of two **non-negative** low-rank matrices:

```
V ≈ W H ,   W (n×r) ≥ 0 ,  H (r×m) ≥ 0 ,  r ≪ min(n, m)
```
- `W`: basis / "parts"; `H`: coefficients/activations. Columns of `V` are **additive combinations** of columns of `W`.
- **Objective:** minimise `‖V − WH‖²_F` (Frobenius norm) or the **KL divergence** `Σ (V log(V/WH) − V + WH)`.
- **Multiplicative update rules (Lee & Seung)** — keep entries non-negative:
  `H ← H ⊙ (WᵀV) / (WᵀWH)` ,  `W ← W ⊙ (VHᵀ) / (WHHᵀ)`
- Solution is **not unique** and the problem is non-convex (local minima); `r` chosen by validation.

**Why NMF:** non-negativity forces a **parts-based, additive, sparse and interpretable** representation (unlike PCA, whose components have mixed signs). 
**Applications:** topic modelling (documents × words → topics), face-image parts, audio source separation, bioinformatics (gene expression), recommender systems.

| | PCA | NMF |
|---|---|---|
| Constraints | Orthogonality | Non-negativity |
| Combination | Additive + subtractive | Only additive |
| Interpretation | Holistic | Parts-based |

### 3.2 Collaborative Filtering (CF)
**Idea:** **Recommender-system** technique that predicts a user's preference for an item using the **preferences of many users** (the crowd), without needing item content. Data = **user–item rating matrix** `R (users × items)` with many **missing** entries.

**Approaches**
| Approach | How |
|---|---|
| **User-based (memory-based)** | Find users similar to the target user (cosine / Pearson similarity); predict rating as the similarity-weighted average of neighbours' ratings. |
| **Item-based** | Find items similar to those the user already liked; recommend those. More stable, widely used. |
| **Model-based / Matrix Factorization (latent factor)** | Factorise `R ≈ U Vᵀ` into user-factor and item-factor matrices; predicted rating `r̂ᵤᵢ = uᵤ · vᵢ`. |

**Matrix-factorisation objective** (over observed ratings only, with L2 regularisation):

```
min_{U,V}  Σ_{(u,i) observed} (rᵤᵢ − uᵤᵀvᵢ)² + λ(‖uᵤ‖² + ‖vᵢ‖²)
```
solved by **SGD** or **Alternating Least Squares (ALS)**. With **non-negativity constraints** this is **NMF-based CF**.

**Similarity measures:** Cosine `= (a·b)/(‖a‖‖b‖)`; Pearson correlation (mean-centred).

**Challenges:** **cold-start** (new user/item), **sparsity**, scalability, popularity bias. Hybrid systems combine CF with content-based filtering.

---

## 4. Mixture of Gaussians (GMM) and Application to Clustering

**Model:** A **probabilistic, soft-clustering** model: data are generated from a weighted sum of `K` Gaussians:

```
p(x) = Σ_{k=1}^{K} π_k · N(x | μ_k, Σ_k) ,    π_k ≥ 0, Σ π_k = 1
```
- `π_k` = mixing coefficient (prior of component k), `μ_k` = mean, `Σ_k` = covariance. Latent variable `z` = which component generated `x`.

**Responsibility** (posterior that component k generated xᵢ — the **soft assignment**):

```
γ_ik = π_k N(xᵢ | μ_k, Σ_k) / Σ_j π_j N(xᵢ | μ_j, Σ_j)
```

**Parameter estimation — EM algorithm** (direct maximum-likelihood has no closed form, so iterate):
1. **Initialise** `π, μ, Σ` (often from K-means).
2. **E-step:** compute responsibilities `γ_ik` with current parameters.
3. **M-step:** re-estimate, with `N_k = Σᵢ γ_ik`:
   - `μ_k = (1/N_k) Σᵢ γ_ik xᵢ`
   - `Σ_k = (1/N_k) Σᵢ γ_ik (xᵢ − μ_k)(xᵢ − μ_k)ᵀ`
   - `π_k = N_k / N`
4. Compute **log-likelihood** `Σᵢ log Σ_k π_k N(xᵢ|μ_k,Σ_k)`; repeat 2–3 until convergence. 
   Log-likelihood never decreases; converges to a local maximum.

**Clustering with GMM:** assign `xᵢ` to `argmax_k γ_ik` (hard) or keep `γ_ik` as soft memberships. Choose `K` using **BIC/AIC**.

| | K-means | GMM (EM) |
|---|---|---|
| Assignment | Hard | **Soft** (probabilities) |
| Cluster shape | Spherical | **Elliptical** (full covariance) |
| Cluster size/density | Assumes equal | Learns `π_k`, `Σ_k` |
| Output | Centroids | Full probability density |

**Applications:** soft clustering, **density estimation**, image segmentation, speaker identification, **anomaly detection** (low `p(x)`), data generation. **Issues:** local optima, singularities (collapse of a component onto one point → regularise `Σ`), needs `K`.

---

## 5. Anomaly Detection

**Definition:** Identify observations that **deviate significantly** from the majority/normal behaviour (**outliers**). The data (training) set **may contain** anomalies — the task is to find them in the data itself (**outlier detection**; typically unsupervised).

**Types of anomalies:** *Point* (single abnormal instance), *Contextual* (abnormal in a context, e.g., a high temperature in winter), *Collective* (a group of instances abnormal together).

**Techniques**
| Category | Idea | Examples |
|---|---|---|
| **Statistical** | Normal data follow a distribution; flag low-probability points | Z-score (`\|z\| > 3`), IQR rule, Gaussian density `p(x) < ε` |
| **Distance-based** | Anomalies are far from neighbours | k-NN distance |
| **Density-based** | Anomalies lie in low-density regions | **LOF** (Local Outlier Factor) |
| **Clustering-based** | Points far from any centroid / in tiny clusters | K-means distance, GMM low likelihood |
| **Isolation-based** | Anomalies are easy to isolate by random splits | **Isolation Forest** (short average path length ⇒ anomaly) |
| **Reconstruction-based** | High reconstruction error | PCA, autoencoders |
| **One-class SVM** | Boundary around normal data | |

**Gaussian-density approach:** fit `p(x)` on the data; flag `x` as anomalous if `p(x) < ε` (ε tuned on a labelled validation set using F1 because anomalies are rare/imbalanced).

**Applications:** fraud detection, intrusion/fault detection, health monitoring, quality control. **Challenges:** class imbalance, defining "normal", evolving behaviour, few labels.

---

## 6. Novelty Detection

**Definition:** Decide whether a **new, unseen** observation belongs to the same distribution as the training data. The training set is assumed to be **clean (contains only normal data)**; the model is then used to detect **novel** patterns in **future** data (**semi-supervised**, one-class learning).

**Anomaly detection vs Novelty detection**
| | Anomaly (outlier) detection | Novelty detection |
|---|---|---|
| Training data | **Contaminated** (contains outliers) | **Clean** (only normal) |
| Goal | Find outliers **in the training data** | Judge **new** observations |
| Learning type | Unsupervised | Semi-supervised (one-class) |
| Methods | Isolation Forest, LOF (outlier mode), Elliptic Envelope | **One-Class SVM**, LOF with `novelty=True`, GMM density threshold, autoencoder |
| Novel points | Often treated as noise | May be a **new class/event** worth learning |

**One-Class SVM:** learns a boundary (hyperplane in kernel space, separating the data from the origin with maximum margin; parameter `ν` ∈ (0,1] bounds the fraction of training errors / support vectors) enclosing the normal data. `f(x) = +1` inside (normal), `−1` outside (novel).

**Applications:** detecting new fault types, new intrusion patterns, new topics in text streams, medical monitoring.

---

## 7. Learning Vector Quantization (LVQ)

**Definition:** A **prototype-based, supervised** classification algorithm introduced by **Teuvo Kohonen**. It learns a small set of **codebook (prototype) vectors**, each labelled with a class; a new point is assigned the **class of its nearest prototype**. Decision regions form a **Voronoi partition**. 
(It derives from unsupervised **Vector Quantization** / competitive learning, and is a precursor to supervised use of SOM ideas — note it uses class labels, so it is *supervised* even though it is taught with unsupervised-style methods.)

**Setup:** choose number of prototypes per class; initialise them (random class samples / K-means centres of each class); learning rate `α(t)` decreasing with time.

**LVQ1 algorithm (winner-take-all)**
1. Pick a training sample `x` with class label `c_x`.
2. Find the **nearest (winner) prototype** `m_w` (Euclidean distance).
3. **Update only the winner:**
   - If class(`m_w`) = `c_x` → **move closer**: `m_w ← m_w + α(t)[x − m_w]`
   - Else → **move away**: `m_w ← m_w − α(t)[x − m_w]`
4. All other prototypes are unchanged.
5. Decrease `α(t)` (e.g., linearly); repeat for many epochs.

**Variants**
- **LVQ2 / LVQ2.1:** update **two** nearest prototypes — the closest of the correct class is moved toward `x` and the closest of the wrong class is moved away — only when `x` falls inside a small **window** near the decision boundary (improves boundary placement).
- **LVQ3:** adds a rule for when both nearest prototypes are of the correct class (stability).
- Generalised LVQ (GLVQ), Robust Soft LVQ (RSLVQ): cost-function based versions.

**Prediction:** `class(x) = class of argmin_m ‖x − m‖`.

| | K-means | KNN | LVQ |
|---|---|---|---|
| Learning | Unsupervised | Supervised (lazy) | Supervised (prototype-based) |
| Stores | K centroids | **All** training points | A **few** learned prototypes |
| Prediction cost | — | High | Low (fast, compact) |

**Pros:** simple, interpretable prototypes, compact model, fast prediction, naturally multi-class. **Cons:** heuristic (LVQ1 has no cost function), depends on initialisation and number of prototypes, sensitive to distance metric/feature scaling, can be unstable on overlapping classes.

---

## Quick Revision Sheet

| Topic | One-line takeaway |
|---|---|
| Unsupervised learning | Find structure in unlabelled data |
| K-means | Assign to nearest centroid ↔ recompute means; minimises WCSS; local optimum |
| Choosing K | Elbow, silhouette |
| Spectral clustering | K-means on K smallest eigenvectors of graph Laplacian `L = D − W`; handles non-convex shapes |
| PCA | Eigenvectors of covariance; max variance; linear |
| Kernel PCA | PCA in kernel-induced feature space; centre `K`; non-linear |
| ICA | Maximise non-Gaussianity to unmix independent sources `x = As`; whiten first |
| NMF | `V ≈ WH`, all ≥ 0; additive parts-based |
| Collaborative filtering | Predict ratings from similar users/items or latent factors `R ≈ UVᵀ` |
| GMM | `Σπ_k N(μ_k, Σ_k)`; fit by EM; soft clustering |
| EM | E: responsibilities γ; M: update π, μ, Σ |
| Anomaly detection | Find outliers in (contaminated) data |
| Novelty detection | Train on clean data; flag new unseen novel points (One-Class SVM) |
| LVQ | Supervised prototypes; winner moves toward same class, away from other class |

## Likely Exam Questions
1. Explain the K-means algorithm. How do you choose K? List its limitations.
2. Explain spectral clustering with the graph Laplacian. When is it better than K-means?
3. Explain PCA step by step. How many components to keep?
4. What is Kernel PCA? How does it differ from PCA?
5. Explain ICA and the cocktail-party problem; compare ICA with PCA.
6. Explain NMF and its update rules. How is it used in recommendation?
7. Explain collaborative filtering (user-based, item-based, matrix factorisation).
8. Explain the Gaussian Mixture Model and the EM algorithm. Compare with K-means.
9. Differentiate anomaly detection and novelty detection with methods.
10. Explain the LVQ1 algorithm and its variants.

---
*Aligned to the syllabus references: ESL — Ch. 13.2 (prototype methods: K-means, LVQ, Gaussian mixtures), Ch. 14.3 (clustering), 14.5 (PCA, kernel PCA, spectral clustering), 14.6 (NMF), 14.7 (ICA); collaborative filtering and anomaly/novelty detection are covered in the supplementary references (ISLR, Bishop, Alpaydin) and standard scikit-learn documentation.*
