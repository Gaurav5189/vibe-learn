# Unit III — Mining Frequent Patterns, Associations & Correlations
### MCA | Data Mining & Data Warehousing | Ravenshaw University

---

## 1. Introduction

**Frequent Pattern Mining** is the process of discovering patterns (itemsets, subsequences, substructures) that appear frequently in a dataset.

**Why it matters:**
- Reveals hidden relationships between items
- Foundation for Association Rule Mining, Correlation Analysis, and Recommendation Systems

**Key Terms:**

| Term | Definition |
|---|---|
| **Itemset** | A collection of one or more items, e.g., {Bread, Butter} |
| **Support (sup)** | Fraction of transactions containing the itemset |
| **Confidence (conf)** | Likelihood that Y is bought given X is bought |
| **Frequent Itemset** | An itemset whose support ≥ minimum support (min_sup) |
| **Association Rule** | Implication of the form X → Y |

---

## 2. Market Basket Analysis

**Concept:** Analyzes what items customers tend to buy *together* in a shopping transaction.

**Example:**

> A supermarket database shows:
> - 80% of customers who buy **Diapers** also buy **Beer**
> - Rule: `{Diapers} → {Beer}`, support = 40%, confidence = 80%

**Business Use Cases:**
- Store shelf placement (keep associated items nearby)
- Cross-selling and product recommendations (Amazon "Frequently Bought Together")
- Promotional bundling

**Transaction Database Example:**

| TID | Items |
|-----|-------|
| T1 | Bread, Milk |
| T2 | Bread, Diapers, Beer, Eggs |
| T3 | Milk, Diapers, Beer, Cola |
| T4 | Bread, Milk, Diapers, Beer |
| T5 | Bread, Milk, Diapers, Cola |

If min_sup = 60% (3 out of 5 transactions):
- {Bread} → sup = 4/5 = 80% ✅
- {Diapers, Beer} → sup = 3/5 = 60% ✅

---

## 3. Frequent Itemset Generation — Apriori Algorithm

### 3.1 The Apriori Principle (Anti-Monotone Property)

> **"If an itemset is infrequent, then all its supersets must also be infrequent."**

This is the key pruning strategy — it avoids checking all 2^n subsets.

**Example:**
If {Bread, Milk} is infrequent → {Bread, Milk, Butter} is automatically infrequent (pruned).

---

### 3.2 Algorithm Steps

```
1. Scan DB → Find frequent 1-itemsets (L1)
2. Generate candidate 2-itemsets (C2) from L1
3. Scan DB → Prune C2 to get L2
4. Repeat until no new frequent itemsets found
```

**Step-by-Step Example:**

Using the transaction table above, min_sup = 60% (≥ 3):

**Pass 1 — 1-itemsets:**

| Itemset | Count | Frequent? |
|---------|-------|-----------|
| {Bread} | 4 | ✅ |
| {Milk} | 4 | ✅ |
| {Diapers} | 4 | ✅ |
| {Beer} | 3 | ✅ |
| {Cola} | 2 | ❌ |
| {Eggs} | 1 | ❌ |

**Pass 2 — 2-itemsets (from L1, excluding Cola & Eggs):**

| Itemset | Count | Frequent? |
|---------|-------|-----------|
| {Bread, Milk} | 3 | ✅ |
| {Bread, Diapers} | 3 | ✅ |
| {Bread, Beer} | 2 | ❌ |
| {Milk, Diapers} | 3 | ✅ |
| {Milk, Beer} | 2 | ❌ |
| {Diapers, Beer} | 3 | ✅ |

**Pass 3 — 3-itemsets (from L2):**

| Itemset | Count | Frequent? |
|---------|-------|-----------|
| {Bread, Milk, Diapers} | 2 | ❌ |
| {Milk, Diapers, Beer} | 2 | ❌ |

No frequent 3-itemsets → **Algorithm stops.**

---

### 3.3 Complexity & Limitations of Apriori

**Weaknesses:**
- Multiple database scans (one per itemset size)
- Generates a huge number of candidate itemsets
- High I/O cost on large datasets

---

## 4. Rule Generation

Once frequent itemsets are found, association rules are generated.

**Formula:**
```
For itemset {A, B, C}, possible rules:
  A → BC,  B → AC,  C → AB
  AB → C,  AC → B,  BC → A
```

**Metrics to evaluate a rule X → Y:**

### Support
```
Support(X → Y) = P(X ∪ Y) = count(X ∪ Y) / Total Transactions
```

### Confidence
```
Confidence(X → Y) = P(Y | X) = Support(X ∪ Y) / Support(X)
```

**Example:**
```
Rule: {Diapers} → {Beer}
Support({Diapers, Beer}) = 3/5 = 60%
Confidence = 60% / 80% = 75%
```
If min_conf = 70% → Rule is accepted ✅

---

## 5. FP-Growth Algorithm (Alternative to Apriori)

### 5.1 Why FP-Growth?

FP-Growth **eliminates candidate generation** and requires only **2 database scans**, making it much faster than Apriori.

**Core Data Structure: FP-Tree (Frequent Pattern Tree)**
- A compact tree representation of transactions
- Shares common prefixes between transactions

---

### 5.2 Algorithm Steps

**Step 1: Scan DB → Find frequent 1-itemsets, sort by frequency (descending)**

Using our example (min_sup = 3):
```
Frequent items (sorted): Bread(4), Milk(4), Diapers(4), Beer(3)
```

**Step 2: Reorder each transaction by frequency:**

| TID | Original | Reordered |
|-----|----------|-----------|
| T1 | Bread, Milk | Bread, Milk |
| T2 | Bread, Diapers, Beer, Eggs | Bread, Diapers, Beer |
| T3 | Milk, Diapers, Beer, Cola | Milk, Diapers, Beer |
| T4 | Bread, Milk, Diapers, Beer | Bread, Milk, Diapers, Beer |
| T5 | Bread, Milk, Diapers, Cola | Bread, Milk, Diapers |

**Step 3: Build FP-Tree by inserting each reordered transaction**

```
root
 └── Bread:4
       ├── Milk:3
       │     ├── Diapers:2
       │     │     └── Beer:1
       │     └── Diapers:1 (continues for T5)
       └── Diapers:1
             └── Beer:1
 └── Milk:1
       └── Diapers:1
             └── Beer:1
```

**Step 4: Mine FP-Tree using Conditional Pattern Bases**

For each frequent item (bottom-up), extract the **conditional pattern base** and build a **conditional FP-tree**.

| Item | Conditional Pattern Base | Conditional FP-Tree | Frequent Patterns |
|------|--------------------------|---------------------|-------------------|
| Beer | {Bread,Milk,Diapers}:1, {Bread,Diapers}:1, {Milk,Diapers}:1 | Diapers:3 | {Diapers,Beer}:3 |
| Diapers | {Bread,Milk}:2, {Bread}:1, {Milk}:1 | Bread:3, Milk:3 | {Bread,Diapers}:3, {Milk,Diapers}:3 |
| Milk | {Bread}:3 | Bread:3 | {Bread,Milk}:3 |

---

### 5.3 Apriori vs FP-Growth

| Feature | Apriori | FP-Growth |
|---------|---------|-----------|
| DB Scans | Multiple (k+1 for k-itemsets) | Only 2 |
| Candidate Generation | Yes (expensive) | No |
| Memory Usage | Lower | Higher (tree structure) |
| Speed | Slower for large data | Much faster |
| Best for | Small datasets | Large, dense datasets |

---

## 6. Evaluation of Association Patterns

High support and confidence alone are **not sufficient** — a rule can be misleading.

### 6.1 Lift

```
Lift(X → Y) = Confidence(X → Y) / Support(Y)
            = P(X ∪ Y) / (P(X) × P(Y))
```

| Lift Value | Interpretation |
|------------|----------------|
| Lift > 1 | X and Y are positively correlated (good rule) |
| Lift = 1 | X and Y are independent |
| Lift < 1 | X and Y are negatively correlated |

**Example:**
```
Confidence({Diapers} → {Beer}) = 75%
Support(Beer) = 60%
Lift = 0.75 / 0.60 = 1.25 → Positive correlation ✅
```

---

### 6.2 Conviction

```
Conviction(X → Y) = (1 - Support(Y)) / (1 - Confidence(X → Y))
```
- High conviction → strong directional relationship
- Conviction = 1 → X and Y are independent

---

### 6.3 The Simpson's Paradox Problem

A trend that appears in separate groups may **disappear or reverse** when groups are combined. This is why evaluation beyond support/confidence is critical.

---

## 7. From Association Analysis to Correlation Analysis

### 7.1 Limitations of Association Rules

Association rules find co-occurrence, **not causation or true correlation.**

**Example of a misleading rule:**
> {Tea} → {Coffee}, confidence = 90%
> But both are popular independently — the rule may just reflect overall popularity.

**Solution: Use correlation measures.**

---

### 7.2 χ² (Chi-Square) Test for Correlation

Used to test **statistical independence** between two itemsets.

```
χ² = Σ [(Observed - Expected)² / Expected]
```

- If χ² is large → items are correlated
- If χ² ≈ 0 → items are independent

**Contingency Table Example:**

|  | Beer | ¬Beer | Total |
|--|------|-------|-------|
| **Diapers** | 3 | 1 | 4 |
| **¬Diapers** | 0 | 1 | 1 |
| **Total** | 3 | 2 | 5 |

```
Expected(Diapers, Beer) = (4 × 3) / 5 = 2.4
χ² = (3-2.4)²/2.4 + ... → computed for all cells
```

---

### 7.3 All-Confidence Measure

```
All-confidence(X) = Support(X) / max_item_support(X)
```
- Ranges between 0 and 1
- Null-invariant (not affected by null transactions)

---

### 7.4 Null-Invariance Problem

Many measures (like χ²) are affected by **null transactions** (transactions containing neither X nor Y), which can be huge in sparse datasets.

**Null-invariant measures (preferred):**
- All-confidence
- Cosine similarity: `cos(A,B) = P(A ∪ B) / √(P(A) × P(B))`
- Kulczynski measure: `(P(A|B) + P(B|A)) / 2`

---

## 8. Quick Revision Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNIT III — QUICK RECAP                        │
├──────────────────────┬──────────────────────────────────────────┤
│ Market Basket        │ Find items bought together               │
│ Frequent Itemset     │ Support ≥ min_sup                        │
│ Association Rule     │ X → Y with support & confidence          │
│ Apriori              │ Level-wise, uses anti-monotone pruning   │
│ FP-Growth            │ Tree-based, 2 scans, no candidates       │
│ Lift                 │ Measures true correlation (>1 = good)    │
│ χ² Test              │ Statistical correlation check            │
│ Null-Invariance      │ Use Cosine/Kulczynski for sparse data    │
└──────────────────────┴──────────────────────────────────────────┘
```

---

## 9. Common Exam Questions

1. **Explain the Apriori algorithm with an example. What is the Apriori principle?**
2. **Compare Apriori and FP-Growth algorithms.**
3. **What is Market Basket Analysis? Give a real-world example.**
4. **Define Support, Confidence, and Lift. How are they computed?**
5. **Explain the FP-Tree construction process with an example.**
6. **What are the drawbacks of using only support and confidence for evaluating association rules?**
7. **What is the null-invariance problem? Name two null-invariant measures.**
8. **How is correlation analysis different from association analysis?**
9. **Explain the χ² measure for correlation with a contingency table.**
10. **What is Simpson's Paradox in the context of association analysis?**

---

*End of Unit III Notes*
