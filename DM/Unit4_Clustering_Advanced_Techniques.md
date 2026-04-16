# Unit IV — Clustering, Visualization & Advanced Techniques
### MCA | Data Mining & Data Warehousing | Ravenshaw University

---

## 1. Clustering Overview

**Clustering** is an **unsupervised learning** technique that groups data points such that:
- Points within the same cluster are **highly similar**
- Points in different clusters are **highly dissimilar**

> Unlike Classification, clustering has **no predefined labels** — the algorithm discovers the groupings itself.

**Real-World Applications:**
| Domain | Example |
|--------|---------|
| Marketing | Customer segmentation |
| Biology | Grouping genes with similar expression |
| Document Analysis | Topic-based news grouping |
| Anomaly Detection | Fraud detection (outlier = unusual cluster) |
| Image Segmentation | Grouping pixels by color/region |

**Types of Clustering:**

```
Clustering Algorithms
├── Partitioning     → K-Means, K-Medoids
├── Hierarchical     → Agglomerative, Divisive
├── Density-Based    → DBSCAN
└── Graph-Based      → Spectral Clustering
```

---

## 2. K-Means Clustering

### 2.1 Concept

Partitions n data points into **k clusters** by minimizing the **within-cluster sum of squares (WCSS)**.

**Objective Function:**
```
Minimize: Σ Σ ||xi - μk||²
          k  xi∈Ck
```
where μk = mean (centroid) of cluster k.

---

### 2.2 Algorithm Steps

```
1. Choose k (number of clusters)
2. Randomly initialize k centroids
3. Assign each point to the nearest centroid
4. Recompute centroids as mean of assigned points
5. Repeat steps 3-4 until centroids don't change (convergence)
```

---

### 2.3 Worked Example

**Dataset:** Points = {2, 4, 10, 12, 3, 20, 30, 11, 25}  
**k = 2**, Initial centroids: C1 = 2, C2 = 4

**Iteration 1 — Assign:**
| Point | Dist to C1(2) | Dist to C2(4) | Cluster |
|-------|--------------|--------------|---------|
| 2 | 0 | 2 | C1 |
| 4 | 2 | 0 | C2 |
| 10 | 8 | 6 | C2 |
| 12 | 10 | 8 | C2 |
| 3 | 1 | 1 | C1 (tie→C1) |
| 20 | 18 | 16 | C2 |
| 30 | 28 | 26 | C2 |
| 11 | 9 | 7 | C2 |
| 25 | 23 | 21 | C2 |

**New centroids:**
- C1 = (2+3)/2 = **2.5**
- C2 = (4+10+12+20+30+11+25)/7 = **16**

Repeat until stable → Typically converges in a few iterations.

---

### 2.4 Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|------------|
| Simple and fast | Must specify k in advance |
| Scales to large datasets | Sensitive to outliers |
| Easy to implement | Assumes spherical clusters |
| | Can converge to local minima |

---

## 3. K-Medoids (PAM — Partitioning Around Medoids)

### 3.1 Concept

Similar to K-Means but uses **actual data points (medoids)** as cluster centers instead of computed means.

> **Medoid** = the most centrally located point in a cluster (minimizes total distance to other points).

**Why K-Medoids over K-Means?**
- **Robust to outliers** — an outlier can drastically shift a mean, but the medoid stays a real data point
- Works well with **non-Euclidean distances** (e.g., categorical data)

### 3.2 PAM Algorithm Steps

```
1. Randomly select k data points as initial medoids
2. Assign each non-medoid point to nearest medoid
3. For each medoid m and non-medoid point o:
   - Compute cost of swapping m with o
   - If swap reduces total cost → make the swap
4. Repeat step 3 until no improvement
```

**Cost Function:**
```
Total Cost = Σ (distance of each point to its medoid)
```

### 3.3 K-Means vs K-Medoids

| Feature | K-Means | K-Medoids |
|---------|---------|-----------|
| Center | Computed mean | Actual data point |
| Outlier sensitivity | High | Low |
| Distance metric | Euclidean | Any metric |
| Speed | Faster | Slower (O(k(n-k)²)) |

---

## 4. Agglomerative Hierarchical Clustering

### 4.1 Concept

Builds a **hierarchy of clusters** using a **bottom-up** approach:
- Start: each point is its own cluster
- Merge the two closest clusters at each step
- Stop: all points in one cluster (or desired k clusters reached)

Result is visualized as a **Dendrogram**.

---

### 4.2 Linkage Criteria (How to measure distance between clusters)

| Linkage | Distance Measured | Behavior |
|---------|------------------|----------|
| **Single** | Min distance between any two points | Elongated, chain-like clusters |
| **Complete** | Max distance between any two points | Compact, roughly equal clusters |
| **Average** | Average of all pairwise distances | Balance between single & complete |
| **Ward's** | Increase in total within-cluster variance | Most commonly used in practice |

---

### 4.3 Step-by-Step Example

**Points:** A(1,1), B(1,2), C(5,5), D(6,5)

**Distance Matrix:**
```
     A    B    C    D
A  [ 0    1   5.7  6.4 ]
B  [ 1    0   5.1  5.8 ]
C  [5.7  5.1   0    1  ]
D  [6.4  5.8   1    0  ]
```

**Step 1:** Min distance = A-B (dist=1) → Merge → {A,B}, C, D  
**Step 2:** Min distance = C-D (dist=1) → Merge → {A,B}, {C,D}  
**Step 3:** Merge {A,B} and {C,D} → {A,B,C,D}

---

### 4.4 Divisive Hierarchical Clustering

**Top-down** approach:
- Start with all points in one cluster
- Recursively split the most heterogeneous cluster
- Less common than agglomerative

---

## 5. DBSCAN (Density-Based Spatial Clustering of Applications with Noise)

### 5.1 Core Idea

Clusters are **dense regions** separated by **sparse regions**. Can find **arbitrarily shaped clusters** and identifies **noise/outliers**.

### 5.2 Key Parameters

| Parameter | Meaning |
|-----------|---------|
| **ε (eps)** | Radius of neighborhood around a point |
| **MinPts** | Minimum points required in ε-neighborhood to form a core point |

### 5.3 Point Classification

```
ε-neighborhood of point p = all points within distance ε from p

Core Point   → |ε-neighborhood(p)| ≥ MinPts
Border Point → In ε-neighborhood of a core point, but not a core itself
Noise Point  → Neither core nor border (outlier)
```

**Visual:**
```
    ●  ●
  ● [C] ●   ← Core point (many neighbors)
    ●  ●
         ○   ← Border point
                  ×   ← Noise (outlier)
```

### 5.4 Algorithm Steps

```
1. Pick an unvisited point p
2. If p is a core point → start a new cluster, expand it:
   - Add all density-reachable points to the cluster
3. If p is not a core point → label as Noise (may become border later)
4. Repeat until all points visited
```

### 5.5 DBSCAN vs K-Means

| Feature | DBSCAN | K-Means |
|---------|--------|---------|
| Cluster shape | Any shape | Spherical |
| Number of clusters | Auto-detected | Must specify k |
| Outlier handling | Explicit noise class | Forced into a cluster |
| Density variation | Struggles with varying density | N/A |

---

## 6. Cluster Evaluation

### 6.1 Internal Evaluation (No ground truth needed)

**Silhouette Score:**
```
s(i) = (b(i) - a(i)) / max(a(i), b(i))

where:
  a(i) = avg distance from point i to other points in SAME cluster
  b(i) = avg distance from point i to points in NEAREST other cluster

Range: [-1, 1]
  +1 → Well clustered
   0 → On cluster boundary
  -1 → Misclassified
```

**Davies-Bouldin Index:**
```
DB = (1/k) Σ max [ (σi + σj) / d(ci, cj) ]
Lower DB = Better clustering
```

### 6.2 External Evaluation (Ground truth available)

| Measure | Description |
|---------|-------------|
| **Purity** | % of dominant class in each cluster |
| **Rand Index** | Agreement between predicted and true clusters |
| **F-measure** | Harmonic mean of precision and recall |
| **NMI** | Normalized Mutual Information |

---

## 7. Graph-Based Clustering

### 7.1 Concept

Represents data as a **graph G = (V, E)** where:
- Nodes = data points
- Edges = similarity between points (weighted)

**Goal:** Find clusters by **cutting edges** with low similarity (Minimum Cut).

### 7.2 Spectral Clustering (Key Method)

```
1. Build similarity matrix W (e.g., Gaussian kernel)
2. Compute Laplacian matrix L = D - W
   (D = diagonal degree matrix)
3. Compute top-k eigenvectors of L
4. Apply K-Means on eigenvector representation
```

**Advantage:** Can find **non-convex, complex-shaped clusters** that K-Means cannot.

---

## 8. Scalable Clustering Algorithms

For **very large datasets**, standard algorithms are too slow.

| Algorithm | Key Idea | Handles |
|-----------|----------|---------|
| **BIRCH** | Builds a CF-Tree (Clustering Feature Tree) to summarize data | Large datasets, single scan |
| **CURE** | Uses multiple representative points per cluster | Outliers, non-spherical shapes |
| **CLARANS** | Randomized version of K-Medoids | Large datasets |
| **Mini-Batch K-Means** | Uses random subsets (mini-batches) per iteration | Very large / streaming data |

**BIRCH — CF (Clustering Feature) Triple:**
```
CF = (N, LS, SS)
N  = Number of data points
LS = Linear Sum of points
SS = Squared Sum of points
```
CF-Trees merge nearby clusters compactly, enabling one-pass clustering.

---

## 9. Visualizing Clusters

### 9.1 Dendrogram

A **tree diagram** showing the order and distance of merges in hierarchical clustering.

```
        ┌─────────────┐
        │             │
    ┌───┴───┐     ┌───┴───┐
    A       B     C       D
```
- **X-axis:** Data points
- **Y-axis:** Distance/dissimilarity at merge
- **Cut the dendrogram** at a height → choose number of clusters

---

### 9.2 Tree Maps

- Represent **hierarchical data** as nested rectangles
- Each rectangle's **area** is proportional to a data value
- Colors represent cluster membership
- Useful for visualizing large datasets in a compact 2D space

**Example:** File system where folder size = rectangle area, subdirectories = nested rectangles

---

### 9.3 Rectangle Plots (Pixel-Oriented Visualization)

- Data dimensions mapped to **rows and columns** of pixels
- Each pixel's **color/intensity** represents a value
- Patterns in colors reveal clusters
- Good for **high-dimensional data**

---

### 9.4 Data Image (Matrix Visualization)

- Entire dataset displayed as an **image/heatmap**
- Rows = data objects, Columns = attributes
- **Reordering rows/columns** by cluster membership reveals block structures
- Similar to a **correlation matrix heatmap**

**Example:**
```
Gene Expression Matrix — after clustering:
Cluster 1 genes │ Cluster 2 genes
────────────────┼────────────────
█████ ░░░░░░░░  │  ░░░░ ██████
█████ ░░░░░░░░  │  ░░░░ ██████
```

---

## 10. Advanced Techniques — Text Mining

### 10.1 What is Text Mining?

Extracting **structured, useful knowledge** from **unstructured text** data (documents, web pages, emails).

**Pipeline:**
```
Raw Text → Preprocessing → Feature Extraction → Mining → Knowledge
```

### 10.2 Text Preprocessing

| Step | Description | Example |
|------|-------------|---------|
| **Tokenization** | Split text into words/tokens | "Data Mining" → ["Data", "Mining"] |
| **Stop word removal** | Remove common uninformative words | Remove "is", "the", "a" |
| **Stemming** | Reduce words to root form | "running" → "run" |
| **Lemmatization** | Reduce to dictionary base form | "better" → "good" |
| **TF-IDF Weighting** | Score word importance | Rare words in a doc get high score |

**TF-IDF Formula:**
```
TF(t,d)  = frequency of term t in document d
IDF(t)   = log(N / df(t))
           N = total docs, df(t) = docs containing t

TF-IDF(t,d) = TF(t,d) × IDF(t)
```

---

### 10.3 Extracting Attributes (Feature Extraction)

**Bag of Words (BoW):** Represent document as a vector of word counts.

```
Doc1: "Data mining is useful"
Doc2: "Mining finds patterns"

Vocabulary: [Data, mining, is, useful, finds, patterns]
Doc1 vector: [1, 1, 1, 1, 0, 0]
Doc2 vector: [0, 1, 0, 0, 1, 1]
```

**N-Grams:** Capture word sequences.
```
Bigrams of "Data Mining is Fun":
"Data Mining", "Mining is", "is Fun"
```

---

### 10.4 Structural Approaches

Go beyond word counts to capture **document structure:**
- **Part-of-Speech (POS) Tagging** — Noun, Verb, Adjective
- **Named Entity Recognition (NER)** — Identify names, places, organizations
- **Dependency Parsing** — Understand grammatical relationships
- **Information Extraction** — Extract facts like (Subject, Relation, Object) triples

**Example NER:**
```
"Apple launched iPhone in California"
 ORG     VERB    PRODUCT    LOCATION
```

---

## 11. Bayesian Approach to Classifying Text

### 11.1 Naive Bayes Text Classifier

Based on **Bayes' Theorem**, assumes features (words) are **conditionally independent** given the class.

**Bayes' Theorem:**
```
P(Class | Document) ∝ P(Class) × Π P(word_i | Class)
```

### 11.2 Example

**Task:** Classify email as Spam or Not Spam.

**Training data:**
```
Spam emails contain: "free", "win", "money" frequently
Ham emails contain: "meeting", "project", "report" frequently
```

**Classifying new email:** "Win free money now"
```
P(Spam | email) ∝ P(Spam) × P("win"|Spam) × P("free"|Spam) × P("money"|Spam)
P(Ham  | email) ∝ P(Ham)  × P("win"|Ham)  × P("free"|Ham)  × P("money"|Ham)
```
If P(Spam|email) > P(Ham|email) → Classify as **Spam** ✅

### 11.3 Laplace Smoothing

To handle words **not seen in training** (zero probability problem):
```
P(word | Class) = (count(word, Class) + 1) / (total_words_in_Class + |Vocabulary|)
```

### 11.4 Advantages of Naive Bayes for Text

- Very fast training and prediction
- Works well even with small training data
- Handles high-dimensional text data well
- Despite "naive" assumption, performs surprisingly well in practice

---

## 12. Web Mining

**Web Mining** = applying data mining techniques to extract knowledge from the **World Wide Web**.

### 12.1 Three Types of Web Mining

```
Web Mining
├── Web Content Mining   → Mine text/images/data on web pages
├── Web Structure Mining → Mine hyperlink structure (graph)
└── Web Usage Mining     → Mine server logs / user behavior
```

---

### 12.2 Classifying Web Pages

**Goal:** Automatically assign categories to web pages (e.g., News, Sports, Technology).

**Approaches:**
1. **Feature-based:** Treat page as text document → apply Naive Bayes / SVM / Decision Trees
2. **Link-based:** Pages linked together are likely in the same category
3. **Hybrid:** Combine content + link structure

**Features used:**
- Words in page title, headings, body
- Anchor text of incoming/outgoing links
- URL structure (e.g., /sports/ in URL)
- Meta tags

---

### 12.3 Extracting Knowledge from the Web

**Key Techniques:**

| Technique | Description |
|-----------|-------------|
| **Web Scraping** | Automated extraction of data from HTML pages |
| **Wrapper Induction** | Learn patterns to extract structured data from semi-structured pages |
| **Information Extraction (IE)** | Extract entities, relationships from unstructured text |
| **PageRank Algorithm** | Rank pages by importance based on link structure |
| **HITS Algorithm** | Identify Hubs (pages with many outlinks) and Authorities (pages with many inlinks) |

**PageRank (simplified):**
```
PR(A) = (1-d) + d × Σ [PR(T) / C(T)]

d = damping factor (typically 0.85)
T = pages linking to A
C(T) = number of outlinks from T
```

**Web Usage Mining:**
- Analyze **server access logs** to find browsing patterns
- Used in: Personalization, recommendation systems, website redesign
- Techniques: Sessionization → Frequent pattern mining on click sequences

---

## 13. Quick Revision Summary

```
┌────────────────────────────────────────────────────────────────────┐
│                      UNIT IV — QUICK RECAP                          │
├──────────────────────────┬─────────────────────────────────────────┤
│ K-Means                  │ Centroid-based, sensitive to outliers   │
│ K-Medoids (PAM)          │ Medoid-based, robust to outliers        │
│ Agglomerative            │ Bottom-up merging, uses dendrogram      │
│ DBSCAN                   │ Density-based, finds noise/any shape    │
│ Graph-Based              │ Spectral clustering, uses eigenvectors  │
│ BIRCH/CURE               │ Scalable for large datasets             │
├──────────────────────────┼─────────────────────────────────────────┤
│ Dendrogram               │ Tree of hierarchical merge order        │
│ Tree Map                 │ Nested rectangles for hierarchy         │
│ Rectangle/Data Image     │ Pixel/heatmap for high-dim data        │
├──────────────────────────┼─────────────────────────────────────────┤
│ Text Mining              │ TF-IDF, BoW, NER, POS tagging          │
│ Naive Bayes (Text)       │ P(Class|Doc) ∝ P(Class)×ΠP(word|Class) │
│ Web Content Mining       │ Classify/extract from page content      │
│ Web Structure Mining     │ PageRank, HITS, link analysis           │
│ Web Usage Mining         │ Log analysis, click-stream patterns     │
└──────────────────────────┴─────────────────────────────────────────┘
```

---

## 14. Common Exam Questions

1. **Explain K-Means clustering with an example. What are its limitations?**
2. **How does K-Medoids differ from K-Means? Why is K-Medoids more robust?**
3. **Explain Agglomerative Hierarchical Clustering. What is a Dendrogram?**
4. **What is DBSCAN? Define Core, Border, and Noise points with an example.**
5. **Compare K-Means and DBSCAN clustering algorithms.**
6. **What is the Silhouette Score? How is it used to evaluate clustering?**
7. **Explain BIRCH or CURE — how do they handle large-scale data?**
8. **What are the different methods to visualize clusters? Explain Dendrogram and Tree Map.**
9. **Explain Text Mining. What is TF-IDF and how is it computed?**
10. **Explain the Naive Bayes approach for text classification with an example.**
11. **What are the three types of Web Mining? Explain each briefly.**
12. **Explain the PageRank algorithm. What is the damping factor?**
13. **What is Information Extraction? Explain NER with an example.**
14. **What is Web Usage Mining? How is it useful in e-commerce?**

---

*End of Unit IV Notes*
