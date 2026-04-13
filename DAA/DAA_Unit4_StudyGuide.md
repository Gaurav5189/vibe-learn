# DAA Unit IV — Complete Study Guide
### MCA | Design and Analysis of Algorithms

---

## TABLE OF CONTENTS
1. [Polynomials & FFT](#1-polynomials--fft)
2. [String Matching](#2-string-matching)
3. [NP-Completeness](#3-np-completeness)
4. [Approximation Algorithms](#4-approximation-algorithms)
5. [Quick Revision Table](#5-quick-revision-table)

---

## 1. Polynomials & FFT

### 1.1 Representation of Polynomials

A polynomial of degree n-1:  
**A(x) = a₀ + a₁x + a₂x² + ... + aₙ₋₁xⁿ⁻¹**

Two representations:

#### i) Coefficient Representation
Store the coefficients as an array:
```
A(x) = 3 + 2x + 5x²   →   [3, 2, 5]
```
- **Addition:** O(n) — add coefficient-wise
- **Multiplication:** O(n²) — convolve two coefficient arrays

#### ii) Point-Value Representation
A polynomial of degree n-1 is **uniquely determined by n point-value pairs**:
```
A(x) = {(x₀, y₀), (x₁, y₁), ..., (xₙ₋₁, yₙ₋₁)}  where yᵢ = A(xᵢ)
```
- **Multiplication:** O(n) — just multiply y-values pointwise!
- But we need matching x-values in both polynomials

**The trick:** Convert coefficient → point-value (Evaluation), multiply, convert back (Interpolation).

```
Coefficient Form ──(FFT: O(n log n))──► Point-Value Form
                                              │
                                        Multiply O(n)
                                              │
Coefficient Form ◄──(Inverse FFT)────── Point-Value Form
```
> This gives **polynomial multiplication in O(n log n)** instead of O(n²)!

---

### 1.2 DFT — Discrete Fourier Transform

**Key Idea:** Choose evaluation points as **complex nth roots of unity**:

```
ωₙ = e^(2πi/n)        (primitive nth root of unity)

The n roots of unity: ωₙ⁰, ωₙ¹, ωₙ², ..., ωₙⁿ⁻¹
```

**DFT of vector a = (a₀, a₁, ..., aₙ₋₁):**
```
yₖ = Σⱼ₌₀ⁿ⁻¹  aⱼ · ωₙʲᵏ    for k = 0, 1, ..., n-1
```
This is exactly evaluating polynomial A(x) at all n roots of unity.

**Properties of Roots of Unity (used in FFT):**
| Property | Statement |
|---|---|
| Halving | (ωₙᵏ)² = ω_{n/2}^k — squaring halves the set |
| Cancellation | ωₙᵏ⁺ⁿ/² = −ωₙᵏ |
| Summation | Σωₙʲᵏ = 0 if k≠0, n if k=0 |

---

### 1.3 FFT — Fast Fourier Transform

**Naive DFT:** O(n²) — too slow.  
**FFT:** O(n log n) — using **divide and conquer**.

**Idea (Cooley-Tukey):** Split polynomial A(x) into even and odd indexed coefficients:
```
A(x) = A_even(x²) + x · A_odd(x²)

A_even(x) = a₀ + a₂x + a₄x² + ...    (even-indexed coefficients)
A_odd(x)  = a₁ + a₃x + a₅x² + ...    (odd-indexed coefficients)
```

**Recursive FFT:**
```
RECURSIVE-FFT(a):
  n = length of a
  if n == 1: return a

  ωₙ = e^(2πi/n),  ω = 1
  a[even] = (a₀, a₂, a₄, ...)
  a[odd]  = (a₁, a₃, a₅, ...)

  y[even] = RECURSIVE-FFT(a[even])   // size n/2
  y[odd]  = RECURSIVE-FFT(a[odd])    // size n/2

  for k = 0 to n/2 - 1:
      y[k]       = y_even[k] + ω · y_odd[k]
      y[k + n/2] = y_even[k] - ω · y_odd[k]
      ω = ω · ωₙ

  return y
```

**Recurrence:** T(n) = 2T(n/2) + O(n) → **T(n) = O(n log n)**

**Inverse FFT (Interpolation):**  
Same algorithm but:
- Replace ωₙ with ωₙ⁻¹ (conjugate)
- Divide final result by n

> **Exam Tip:** You don't need to derive FFT from scratch. Know the divide-and-conquer idea, the recurrence, and that it runs in O(n log n). The key insight is splitting into even/odd halves.

**Summary of Polynomial Multiplication via FFT:**
```
Step 1: FFT(A) and FFT(B)      — O(n log n)
Step 2: Pointwise multiply      — O(n)
Step 3: Inverse FFT             — O(n log n)
Total:  O(n log n)
```

---

## 2. String Matching

**Problem:** Given text T[1..n] and pattern P[1..m], find all occurrences of P in T.

---

### 2.1 Naïve String-Matching Algorithm

**Idea:** Try every possible starting position in T, check if P matches.

```
NAIVE-STRING-MATCHER(T, P):
  n = length(T)
  m = length(P)
  for s = 0 to n - m:          // s = shift
      if P[1..m] == T[s+1..s+m]:
          print "Pattern occurs at shift s"
```

**Time Complexity:** **O((n - m + 1) · m)** = O(nm) in worst case

**Example:**
```
T = "AAAAAB"    (n=6)
P = "AAAB"      (m=4)

s=0: AAAA vs AAAB → mismatch at position 4
s=1: AAAA vs AAAB → mismatch at position 4
s=2: AAAB vs AAAB → MATCH at shift 2!

Worst case: T="AAAA...A", P="AAA...AB" → O(nm) comparisons
```

**Drawback:** Doesn't use info from previous comparisons — very slow in worst case.

---

### 2.2 Rabin-Karp Algorithm

**Idea:** Use **hashing** to compare the pattern with substrings of text in O(1) (instead of character-by-character).

**Hash Function (Horner's method):**
```
h(s) = (s[1]·d^(m-1) + s[2]·d^(m-2) + ... + s[m]) mod q

where d = number of characters (alphabet size, e.g. 256)
      q = a prime number (to reduce collisions)
```

**Rolling Hash (Sliding Window):**  
When window slides by 1, update hash in O(1):
```
t(s+1) = (d · (t(s) − T[s+1] · h) + T[s+m+1]) mod q

where h = d^(m-1) mod q   (precomputed)
```

**Algorithm:**
```
RABIN-KARP(T, P, d, q):
  n = length(T), m = length(P)
  h = d^(m-1) mod q
  p_hash = 0, t_hash = 0

  for i = 1 to m:        // compute hash of P and first window of T
      p_hash = (d·p_hash + P[i]) mod q
      t_hash = (d·t_hash + T[i]) mod q

  for s = 0 to n-m:
      if p_hash == t_hash:                  // hash match
          if P[1..m] == T[s+1..s+m]:       // verify (avoid spurious hits)
              print "Pattern at shift s"
      if s < n-m:
          t_hash = (d·(t_hash − T[s+1]·h) + T[s+m+1]) mod q

```

**Complexity:**
| Case | Time |
|---|---|
| Best/Average | O(n + m) |
| Worst Case | O(nm) — if all hashes match (spurious hits) |

**Spurious Hit:** Hash values match but actual strings don't. Happens due to mod operation.

> **Exam Tip:** Rabin-Karp is best when searching for **multiple patterns** simultaneously — compute all pattern hashes and match against rolling window hash. Also useful for plagiarism detection.

**Comparison:**
| | Naïve | Rabin-Karp |
|---|---|---|
| Preprocessing | O(1) | O(m) |
| Matching | O(nm) | O(n+m) avg |
| Worst Case | O(nm) | O(nm) |
| Key Idea | Brute force | Rolling hash |

---

## 3. NP-Completeness

### 3.1 Complexity Classes

**Decision Problem:** A problem with YES/NO answer.
*(NP theory is defined for decision problems)*

#### Class P
Problems **solvable in polynomial time** by a deterministic algorithm.
```
Examples: Sorting, BFS/DFS, Shortest Path (Dijkstra), MST
Time: O(nᵏ) for some constant k
```

#### Class NP
Problems **verifiable in polynomial time** — given a **certificate** (candidate solution), we can verify it's correct in polynomial time.
```
Examples: Hamiltonian Cycle, SAT, Subset Sum, Clique
Key: Verification is easy, but finding solution may be hard
```

> **Important:** P ⊆ NP (anything solvable in poly time is also verifiable in poly time). Whether P = NP is the **greatest unsolved problem in computer science**.

#### Class NP-Hard
Problems that are **at least as hard as the hardest NP problems**.
- Every NP problem can be reduced to it.
- May or may not be in NP.

#### Class NP-Complete
Problems that are **both NP and NP-Hard**.
```
NP-Complete = NP ∩ NP-Hard
```

```
         ┌─────────────────────────────┐
         │            NP               │
         │    ┌──────────────┐         │
         │    │      P       │         │
         │    └──────────────┘         │
         │                  ┌────────┐ │
         │                  │  NPC   │ │
         │                  └────────┘ │
         └─────────────────────────────┘
                              ↕
                           NP-Hard (outside NP too)
```

---

### 3.2 Polynomial-Time Verification

A **verification algorithm** takes:
- Input: problem instance x
- Certificate: y (proposed solution)
- Outputs: YES if y is a valid solution for x

**Example — Hamiltonian Cycle:**
- Finding: Is there a cycle visiting all vertices? → Hard (NP)
- Verifying: Given a sequence of vertices, check if it's a valid Hamiltonian cycle → Easy O(V)

**Language L ∈ NP** if ∃ polynomial-time verification algorithm for L.

---

### 3.3 Reducibility

**Polynomial-time reducibility (≤ₚ):**
> Problem A reduces to Problem B (written A ≤ₚ B) if any instance of A can be transformed into an instance of B in polynomial time.

```
A ≤ₚ B means:  "B is at least as hard as A"
               "If B is easy, then A is easy"
               "If A is hard, then B is hard"
```

**Proving NP-Completeness:**
To show problem B is NP-Complete:
1. Show B ∈ NP (give a polynomial-time verifier)
2. Show B is NP-Hard: take a **known NP-Complete problem A** and show **A ≤ₚ B**

```
Known NPC → (reduction) → New Problem
If reduction is poly-time, new problem is also NP-Hard
```

---

### 3.4 NP-Completeness Proofs

**The First NP-Complete Problem — SAT (Cook's Theorem, 1971):**
> Boolean Satisfiability (SAT) is NP-Complete.

SAT: Given a Boolean formula, is there an assignment of TRUE/FALSE to variables that makes the formula TRUE?
```
Example: (x₁ ∨ ¬x₂) ∧ (¬x₁ ∨ x₃) ∧ (x₂ ∨ ¬x₃)
Is there x₁, x₂, x₃ ∈ {T,F} that satisfies this? → YES: x₁=T, x₂=F, x₃=T
```

**3-CNF-SAT:** Each clause has exactly 3 literals. Also NP-Complete.
*(SAT ≤ₚ 3-CNF-SAT)*

**Reduction Chain (important for exams):**
```
SAT
 └─► 3-CNF-SAT
       ├─► CLIQUE
       ├─► VERTEX COVER
       ├─► INDEPENDENT SET
       ├─► SUBSET SUM
       └─► HAMILTONIAN CYCLE
             └─► TRAVELLING SALESMAN (decision version)
```

---

### 3.5 Common NP-Complete Problems

| Problem | Description |
|---|---|
| **SAT** | Boolean formula satisfiability |
| **3-CNF-SAT** | SAT with clauses of exactly 3 literals |
| **CLIQUE** | Does graph G have a clique of size k? |
| **Vertex Cover** | Is there a vertex cover of size ≤ k? |
| **Independent Set** | Is there an independent set of size ≥ k? |
| **Hamiltonian Cycle** | Does graph have a cycle visiting all vertices exactly once? |
| **TSP (decision)** | Is there a tour of cost ≤ k visiting all cities? |
| **Subset Sum** | Does any subset of integers sum to exactly W? |
| **Set Cover** | Can k sets cover all elements? |

> **Exam Tip:** Clique, Vertex Cover, and Independent Set are **complement problems** of each other — knowing one helps reduce to another. G has a clique of size k ⟺ complement(G) has an independent set of size k.

---

## 4. Approximation Algorithms

Since NP-Complete problems can't be solved optimally in polynomial time (unless P=NP), we use **approximation algorithms** — run in polynomial time and produce solutions **close to optimal**.

**Approximation Ratio ρ(n):**
```
For minimization: C/C* ≤ ρ(n)
For maximization: C*/C ≤ ρ(n)

C  = cost of approximate solution
C* = cost of optimal solution
ρ(n) ≥ 1  (closer to 1 = better approximation)
```

---

### 4.1 Vertex Cover Problem

**Problem:** Find the smallest set of vertices such that **every edge** has at least one endpoint in the set.
*(NP-Hard to solve optimally)*

**Greedy Approximation (2-approximation):**
```
APPROX-VERTEX-COVER(G):
  C = ∅
  E' = copy of G.E
  while E' is not empty:
      (u, v) = any edge from E'
      C = C ∪ {u, v}              // add BOTH endpoints
      remove all edges incident to u or v from E'
  return C
```

**Approximation Ratio: 2** → Solution is at most **2× optimal**

**Why 2?**
- Let A = set of edges picked by algorithm (they form a matching — no two share a vertex)
- Every edge in A must be covered → |C*| ≥ |A|
- Our cover C adds 2 vertices per edge → |C| = 2|A| ≤ 2|C*|

**Example:**
```
Graph: 1-2, 2-3, 3-4, 4-5, 5-6

Pick edge (1,2) → add {1,2}, remove edges of 1 and 2
Pick edge (3,4) → add {3,4}, remove edges of 3 and 4
Pick edge (5,6) → add {5,6}

C = {1,2,3,4,5,6}  (size 6)
Optimal = {2,3,4,5} (size 4)
Ratio = 6/4 = 1.5 ≤ 2 ✓
```

---

### 4.2 Travelling Salesman Problem (TSP)

**Problem:** Find the minimum-cost tour that visits all cities exactly once and returns to start.

**Two versions:**
1. **General TSP:** No approximation possible within any constant ratio (unless P=NP)
2. **TSP with Triangle Inequality:** c(u,w) ≤ c(u,v) + c(v,w) — has a **2-approximation**

**2-Approximation for Metric TSP:**
```
APPROX-TSP-TOUR(G, c):
  r = any vertex as root
  T* = Minimum Spanning Tree of G (using Prim's or Kruskal's)
  L  = list of vertices in preorder walk of T*  // visit order via DFS
  return Hamiltonian cycle in order of L
```

**Why it works:**
- Optimal TSP tour C* ≥ MST weight (removing any edge gives a spanning tree)
- Full walk of MST = 2 · MST weight (each edge traversed twice)
- Shortcutting via triangle inequality doesn't increase cost
- So cost of tour ≤ 2 · MST ≤ 2 · C*

**Christofides Algorithm:** Achieves **3/2-approximation** for metric TSP (better in practice).

---

### 4.3 Set-Covering Problem

**Problem:** Given universe U and family of sets S = {S₁, S₂, ..., Sₖ}, find the **minimum number of sets** whose union covers U.

**Greedy Approximation:**
```
GREEDY-SET-COVER(U, S):
  C = ∅
  while U ≠ ∅:
      pick Sᵢ ∈ S that covers the most uncovered elements
      C = C ∪ {Sᵢ}
      U = U - Sᵢ
  return C
```

**Approximation Ratio: H(max|Sᵢ|)** = **O(log n)**
*(where H(d) = 1 + 1/2 + 1/3 + ... + 1/d is the harmonic series)*

**Example:**
```
U = {1,2,3,4,5,6,7}
S₁={1,2,3,4}, S₂={2,4,5,6}, S₃={1,4,7}, S₄={3,5,6,7}

Round 1: S₁ covers 4 elements → pick S₁. Remaining: {5,6,7}
Round 2: S₂ covers {5,6}, S₄ covers {5,6,7} → pick S₄. Remaining: {}
Done!

C = {S₁, S₄}  (size 2)
```

**Why O(log n)?**
After k greedy selections, at most n·(1 - 1/opt)^k elements remain.
Total rounds ≤ opt · ln(n) = **O(opt · log n)**

---

### 4.4 Subset-Sum Problem

**Problem:** Given a set S = {x₁, x₂, ..., xₙ} of positive integers and target t, does any subset sum to exactly t?

**Decision version:** NP-Complete  
**Optimization version:** Find subset with sum as close to t as possible without exceeding it.

**Exact Algorithm (DP):**
```
Build table L[i][w] = TRUE if subset of {x₁,...,xᵢ} sums to w

L[i][w] = L[i-1][w]  OR  (w ≥ xᵢ AND L[i-1][w-xᵢ])

Time: O(n·t)  — pseudo-polynomial (exponential in input size bits)
```

**Approximation Scheme (FPTAS):**  
For approximation ratio (1+ε):
1. **Trim** the list by removing elements too close to each other
   - Trim by factor δ = εt/n
2. Run exact DP on trimmed list
3. Return best sum ≤ t

**Time:** O(n²/ε) — **Fully Polynomial-Time Approximation Scheme (FPTAS)**

**Example of trimming:**
```
L = {10, 11, 12, 15, 20, 21, 22}  with δ = 2
Keep only elements where next element > (1+δ) × previous:
Trimmed: {10, 12, 15, 20, 22}
```

> **Exam Tip:** Subset Sum has a **PTAS and FPTAS** — this makes it "easier" than TSP (general) among NP-hard problems. Know the difference between PTAS and FPTAS.

**PTAS vs FPTAS:**
| | PTAS | FPTAS |
|---|---|---|
| Full form | Poly-Time Approx Scheme | Fully Poly-Time Approx Scheme |
| Running time | poly in n, may be exp in 1/ε | poly in both n AND 1/ε |
| Better? | — | Yes, strictly better |
| Example | TSP (metric) | Subset-Sum, Knapsack |

---

## 5. Quick Revision Table

| Topic | Algorithm/Concept | Key Idea | Complexity |
|---|---|---|---|
| **Polynomial Multiply** | FFT | DFT at roots of unity, divide & conquer | O(n log n) |
| **DFT** | Coefficient → Point-Value | Evaluate at nth roots of unity | O(n²) naïve |
| **String Matching** | Naïve | Check every shift | O(nm) |
| **String Matching** | Rabin-Karp | Rolling hash comparison | O(n+m) avg |
| **NP** | Complexity Class | Verifiable in poly time | — |
| **P** | Complexity Class | Solvable in poly time | — |
| **NP-Complete** | NP ∩ NP-Hard | Hardest problems in NP | — |
| **Reduction** | A ≤ₚ B | Transform A to B in poly time | — |
| **First NPC** | SAT (Cook 1971) | Boolean formula satisfiability | — |
| **Vertex Cover** | 2-Approx Greedy | Pick both endpoints of any edge | O(V+E), ratio=2 |
| **TSP (metric)** | MST-based | Preorder walk of MST | O(E log V), ratio=2 |
| **Set Cover** | Greedy | Always pick set covering most new elements | O(n log n) ratio |
| **Subset Sum** | DP + Trimming | FPTAS with trimmed lists | O(n²/ε) |

---

## Key Exam Points to Remember

1. **FFT:** Splits polynomial into even/odd halves recursively. Recurrence T(n) = 2T(n/2) + O(n) → O(n log n). Uses nth roots of unity as evaluation points.

2. **Rabin-Karp:** Rolling hash avoids recomputing hash from scratch. Spurious hits occur when hashes match but strings don't — verify with direct comparison. Works in O(n+m) on average.

3. **P vs NP:** P = solvable in poly time. NP = verifiable in poly time. P ⊆ NP. Whether P=NP is unsolved.

4. **NP-Complete:** Must show (i) problem is in NP, and (ii) a known NPC problem reduces to it in poly time.

5. **Reduction direction:** If A ≤ₚ B and B ∈ P, then A ∈ P. Equivalently, if A is NP-Hard and A ≤ₚ B, then B is NP-Hard.

6. **Cook's Theorem:** SAT was the FIRST problem proven NP-Complete (1971). Everything else was shown NPC by reducing FROM SAT.

7. **Approximation ratios to remember:**
   - Vertex Cover → **2**
   - TSP (metric) → **2** (Christofides → 3/2)
   - Set Cover → **H(d) ≈ ln n**
   - Subset Sum → **(1+ε)** via FPTAS

8. **General TSP has NO polynomial-time constant approximation** — unless P=NP. (Triangle inequality must hold for the 2-approx to work.)

9. **Subset Sum DP** runs in O(nt) — this is **pseudo-polynomial** because t can be exponentially large in its bit representation.

---

*End of Unit IV — All the Best! 🎯*
