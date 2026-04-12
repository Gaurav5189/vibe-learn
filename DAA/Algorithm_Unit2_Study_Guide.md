# Algorithm Design & Analysis - Unit II Study Guide
## Dynamic Programming & Greedy Algorithms
### Exam Format: Explanation + Pseudocode + Time Complexity Analysis

---

## PART 1: DYNAMIC PROGRAMMING FUNDAMENTALS

### 1.1 What is Dynamic Programming?

Dynamic Programming (DP) is an algorithmic technique that solves complex problems by:
1. Breaking them into overlapping subproblems
2. Solving each subproblem only once
3. Storing results to avoid recomputation (memoization)
4. Building solutions from bottom-up

**When to Use DP:**
- Problem has **overlapping subproblems** (same subproblems solved repeatedly)
- Problem exhibits **optimal substructure** (optimal solution built from optimal subproblems)

**Example:** Fibonacci
- Naive recursion: Solves fib(5) many times → Exponential time O(2ⁿ)
- DP approach: Store fib(5) once → Linear time O(n)

### 1.2 Two Approaches: Top-Down vs Bottom-Up

**Top-Down (Memoization):**
- Write recursive solution
- Store results in hash table/array
- Check cache before computing

```
FIBONACCI-MEMO(n, memo):
    if n in memo
        return memo[n]
    if n ≤ 1
        return n
    memo[n] = FIBONACCI-MEMO(n-1, memo) + FIBONACCI-MEMO(n-2, memo)
    return memo[n]

Time: O(n), Space: O(n)
```

**Bottom-Up (Tabulation):**
- Build table iteratively from base cases
- Fill table in order of dependency

```
FIBONACCI-TAB(n):
    if n ≤ 1
        return n
    dp[0] = 0
    dp[1] = 1
    for i = 2 to n
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

Time: O(n), Space: O(n) [can be O(1) with optimization]
```

---

## PART 2: MATRIX-CHAIN MULTIPLICATION

### 2.1 The Problem

Given a sequence of matrices A₁, A₂, ..., Aₙ, fully parenthesize their product to minimize scalar multiplications.

**Key Insight:** Matrix multiplication is associative. (AB)C = A(BC) give same result but different multiplication costs!

**Example:**
```
A: 10×30    B: 30×5    C: 5×60

(AB)C = (10×30×5) + (10×5×60) = 1500 + 3000 = 4500 multiplications
A(BC) = (30×5×60) + (10×30×60) = 9000 + 18000 = 27000 multiplications

First way is 6× better!
```

### 2.2 Recursive Structure

Let m[i,j] = minimum cost to multiply matrices from i to j

**Optimal Substructure:**
If we split at k: (A_i...A_k)(A_{k+1}...A_j)

Cost = m[i,k] + m[k+1,j] + cost of multiplying results

Result of A_i...A_k is p_{i-1} × p_k
Result of A_{k+1}...A_j is p_k × p_j
Multiplying them costs: p_{i-1} × p_k × p_j

```
m[i,j] = min(m[i,k] + m[k+1,j] + p[i-1]×p[k]×p[j]) 
         for i ≤ k < j
```

Base case: m[i,i] = 0 (single matrix, no multiplication)

### 2.3 Algorithm

```
MATRIX-CHAIN-ORDER(p)
    n = length(p) - 1
    
    // m[i,j] = min cost for multiplying matrices i to j
    // s[i,j] = k value that gives minimum for m[i,j]
    
    for i = 1 to n
        m[i,i] = 0
    
    // l is chain length (2 to n)
    for l = 2 to n
        for i = 1 to n-l+1
            j = i + l - 1
            m[i,j] = ∞
            
            for k = i to j-1
                cost = m[i,k] + m[k+1,j] + p[i-1]×p[k]×p[j]
                
                if cost < m[i,j]
                    m[i,j] = cost
                    s[i,j] = k
    
    return m, s

Time Complexity: O(n³)
Space Complexity: O(n²)

Why O(n³)?
- Three nested loops: O(n) × O(n) × O(n)
- Innermost loop finds minimum over k values
- Total: n(n-1)(n-2)/6 ≈ n³/6 = O(n³)
```

### 2.4 Complete Example

**Matrices:** A₁(10×30), A₂(30×5), A₃(5×60)

**Array p = [10, 30, 5, 60]** (p[i-1] × p[i] is dimension of A_i)

**Step 1: Base cases (length 1)**
```
m[1,1] = 0
m[2,2] = 0
m[3,3] = 0
```

**Step 2: Chain length 2**
```
m[1,2] = m[1,1] + m[2,2] + p[0]×p[1]×p[2]
       = 0 + 0 + 10×30×5 = 1500, s[1,2] = 1

m[2,3] = m[2,2] + m[3,3] + p[1]×p[2]×p[3]
       = 0 + 0 + 30×5×60 = 9000, s[2,3] = 2
```

**Step 3: Chain length 3**
```
m[1,3] = min(
    m[1,1] + m[2,3] + p[0]×p[1]×p[3] = 0 + 9000 + 10×30×60 = 27000
    m[1,2] + m[3,3] + p[0]×p[2]×p[3] = 1500 + 0 + 10×5×60 = 4500  ← MIN
)
s[1,3] = 2
```

**Final Table:**
```
     1      2      3
1    0     1500   4500
2           0     9000
3                  0
```

**Optimal Cost:** m[1,3] = 4500
**Parenthesization:** (A₁A₂)A₃ because s[1,3] = 2

### 2.5 Reconstruction (Building the Solution)

```
PRINT-OPTIMAL-PARENS(s, i, j):
    if i == j
        print "A" + i
    else
        print "("
        PRINT-OPTIMAL-PARENS(s, i, s[i,j])
        PRINT-OPTIMAL-PARENS(s, s[i,j]+1, j)
        print ")"

// For our example:
// PRINT-OPTIMAL-PARENS(s, 1, 3) outputs: ((A₁A₂)A₃)
```

---

## PART 3: ELEMENTS OF DYNAMIC PROGRAMMING

### 3.1 Optimal Substructure

A problem exhibits optimal substructure if an optimal solution can be constructed from optimal solutions of its subproblems.

**Check for Optimal Substructure:**
1. Identify subproblems
2. Assume optimal solution to subproblems
3. Show you can construct optimal solution to original problem
4. Verify no dependency contradictions

**Examples:**
- **Matrix chain:** Optimal solution to MCM(i,j) uses optimal solutions of MCM(i,k) and MCM(k+1,j) ✓
- **Longest path in DAG:** Optimal path A→D uses optimal paths A→B→D ✓
- **Shortest path:** Works for non-negative weights, fails for negative cycles ✗

### 3.2 Overlapping Subproblems

A problem has overlapping subproblems if the recursive algorithm solves the same subproblems repeatedly.

**Example: Fibonacci**
```
fib(5) calls:
          fib(5)
        /      \
     fib(4)    fib(3)
    /    \      /    \
fib(3)  fib(2) fib(2) fib(1)
  / \    / \    / \
fib(2) fib(1)...

Notice: fib(3) appears twice, fib(2) appears thrice!
Naive: O(2ⁿ)
With DP: O(n)
```

**Proof:** Recursive tree has exponential depth but polynomial distinct subproblems.

### 3.3 Memoization vs Tabulation Trade-offs

| Aspect | Memoization | Tabulation |
|--------|-------------|-----------|
| Approach | Top-down recursive | Bottom-up iterative |
| Intuitive | YES - write recursion | NO - need order of computation |
| Code | Simpler | More complex initialization |
| Efficiency | Same asymptotic | Same asymptotic |
| Space | Extra recursion stack | Just DP table |
| Lazy Computation | Only computes needed | Computes all |

**Choose memoization when:** Problem is recursive-friendly or you don't need all subproblems
**Choose tabulation when:** You want guaranteed iterative solution or tight space control

---

## PART 4: LONGEST COMMON SUBSEQUENCE (LCS)

### 4.1 Problem Definition

Given two sequences X and Y, find the longest sequence of characters that appear in both X and Y in the same order (but not necessarily contiguous).

**Example:**
```
X = "AGGTAB"
Y = "GXTXAYB"
LCS = "GTAB" (length 4)

Subsequences don't need to be contiguous:
X: A G G T A B
Y: G X T X A Y B
     ↑    ↑ ↑   ↑
```

**Applications:**
- DNA sequence alignment
- Diff tools (find differences in files)
- Plagiarism detection
- Version control systems

### 4.2 Optimal Substructure

Let X = x₁x₂...xₘ and Y = y₁y₂...yₙ

**Case 1:** If xₘ = yₙ
- Then LCS(X,Y) = LCS(x₁...xₘ₋₁, y₁...yₙ₋₁) + xₘ

**Case 2:** If xₘ ≠ yₙ
- Then LCS(X,Y) = longer of:
  - LCS(x₁...xₘ₋₁, y₁...yₙ)
  - LCS(x₁...xₘ, y₁...yₙ₋₁)

**Recurrence:**
```
       0                           if i=0 or j=0
c[i,j] = c[i-1,j-1] + 1           if x[i] == y[j]
       max(c[i-1,j], c[i,j-1])   if x[i] ≠ y[j]
```

### 4.3 Algorithm - Bottom-Up DP

```
LCS-LENGTH(X, Y):
    m = length(X)
    n = length(Y)
    
    // c[i,j] = length of LCS of X[1..i] and Y[1..j]
    // b[i,j] = pointer for reconstruction
    
    for i = 0 to m
        c[i,0] = 0
    for j = 0 to n
        c[0,j] = 0
    
    for i = 1 to m
        for j = 1 to n
            if X[i] == Y[j]
                c[i,j] = c[i-1,j-1] + 1
                b[i,j] = "↖" (diagonal)
            else
                if c[i-1,j] >= c[i,j-1]
                    c[i,j] = c[i-1,j]
                    b[i,j] = "↑" (up)
                else
                    c[i,j] = c[i,j-1]
                    b[i,j] = "←" (left)
    
    return c, b

Time Complexity: O(m×n)
Space Complexity: O(m×n) for table, O(min(m,n)) if not storing path
```

### 4.4 Complete Example

**X = "ABCD", Y = "ACBD"**

**Step 1: Initialize**
```
       ""  A  C  B  D
    "" 0   0  0  0  0
    A  0   
    B  0   
    C  0   
    D  0   
```

**Step 2: Fill table (first row: A)**
```
X[1]='A' vs Y[1]='A': MATCH → c[1,1] = c[0,0]+1 = 1
X[1]='A' vs Y[2]='C': NO MATCH → c[1,2] = max(c[0,2], c[1,1]) = 1
X[1]='A' vs Y[3]='B': NO MATCH → c[1,3] = max(c[0,3], c[1,2]) = 1
X[1]='A' vs Y[4]='D': NO MATCH → c[1,4] = max(c[0,4], c[1,3]) = 1

       ""  A  C  B  D
    "" 0   0  0  0  0
    A  0   1  1  1  1
    B  0   
    C  0   
    D  0   
```

**Step 3: Fill remaining rows**
```
After filling all:

       ""  A  C  B  D
    "" 0   0  0  0  0
    A  0   1  1  1  1
    B  0   1  1  2  2
    C  0   1  2  2  2
    D  0   1  2  2  3

c[4,4] = 3, so LCS length = 3
```

**Step 4: Reconstruction**

```
PRINT-LCS(b, X, i, j):
    if i == 0 or j == 0
        return
    if b[i,j] == "↖"
        PRINT-LCS(b, X, i-1, j-1)
        print X[i]
    else if b[i,j] == "↑"
        PRINT-LCS(b, X, i-1, j)
    else
        PRINT-LCS(b, X, i, j-1)

// Calling PRINT-LCS(b, X, 4, 4) gives: "ABD"
```

### 4.5 Properties of LCS

- **Symmetry:** LCS(X,Y) = LCS(Y,X)
- **Multiple LCS possible:** X="AB", Y="BA" → LCS can be "A" or "B"
- **Subadditivity:** LCS(X,Y) ≤ min(|X|, |Y|)
- **Connection to edit distance:** Edit distance = m + n - 2×LCS(m,n) for insertions/deletions

---

## PART 5: GREEDY ALGORITHMS FUNDAMENTALS

### 5.1 What is a Greedy Algorithm?

A greedy algorithm makes locally optimal choices at each step, hoping to find a global optimum.

**Greedy Strategy:**
1. Make the locally best choice now
2. Never reconsider the choice
3. Hope it leads to global optimum

**When Greedy Works:**
- Problem has optimal substructure
- Problem exhibits the "greedy choice property"

**Greedy Choice Property:**
A globally optimal solution can be arrived at by making locally optimal (greedy) choices.

**Caution:** Greedy doesn't always work! (e.g., longest path in general graphs)

### 5.2 Proving Greedy Correctness

**Standard Proof Technique:**

1. **Show optimal substructure exists**
2. **Show greedy choice is safe:**
   - Assume optimal solution with different first choice
   - Replace first choice with greedy choice
   - Prove this is still optimal
3. **Show subproblem is similar**
4. **Induct on remaining problem**

---

## PART 6: ACTIVITY-SELECTION PROBLEM

### 6.1 Problem Definition

Given a set of activities, each with start time s[i] and finish time f[i], select maximum number of non-overlapping activities.

**Example:**
```
Activity  1   2   3   4   5   6  7
Start     1   3   0   5   3   5  6
Finish    4   5   6   7   8   9  10

Non-overlapping means: if activity i is selected, no other 
activity with start time in [s[i], f[i]) can be selected.
```

### 6.2 Greedy Strategy

**Key Insight:** Always select activity that finishes earliest!

**Intuition:** By choosing the activity that finishes earliest, you leave the most room for remaining activities.

**Pseudocode:**

```
ACTIVITY-SELECTION(s, f):
    n = length(s)
    
    // Sort by finish time (assume already sorted)
    activities = [(1, s[1], f[1]), ...]
    
    // Start with first activity (earliest finish)
    A = {1}
    k = 1  // index of last selected activity
    
    for i = 2 to n
        if s[i] >= f[k]  // activity i starts after k finishes
            A = A ∪ {i}
            k = i
    
    return A

Time Complexity: O(n log n) for sorting + O(n) for selection = O(n log n)
Space Complexity: O(n)
```

### 6.3 Complete Example

**Activities sorted by finish time:**
```
Activity  1   2   3   4   5
Start     1   3   0   5   8
Finish    4   5   6   7   10
```

**Execution:**
```
Step 1: Select activity 1 (0,1,4)
        A = {1}, last_finish = 4

Step 2: Check activity 2: start=3 < 4 → SKIP
Step 3: Check activity 3: start=0 < 4 → SKIP
Step 4: Check activity 4: start=5 >= 4 → SELECT
        A = {1,4}, last_finish = 7

Step 5: Check activity 5: start=8 >= 7 → SELECT
        A = {1,4,5}, last_finish = 10

Return: {1,4,5} (3 activities selected)
```

### 6.4 Proof of Greedy Correctness

**Theorem:** The greedy algorithm selects a maximum set of compatible activities.

**Proof:**
Let A be the greedy solution, O be any optimal solution.

Let a₁, a₂, ..., aₖ be activities in A (sorted by finish time).
Let o₁, o₂, ..., oₘ be activities in O (sorted by finish time).

**Claim:** k = m (greedy is optimal)

**Proof by Induction:**

Base case: a₁ = o₁
- Greedy picks activity with earliest finish
- This is no worse than any first choice in O
- Can replace o₁ with a₁ to get O' that's still optimal

Inductive step: If A[1..i] matches O[1..i], then greedy choice for position i+1 is optimal.
- Same argument: earliest finish leaves most room
- Can always replace o_{i+1} with a_{i+1}

Conclusion: Greedy solution equals or exceeds optimal solution.
Since we can't exceed optimal, greedy IS optimal. ∎

### 6.5 Why Greedy Can Fail with Wrong Strategy

**Earliest Start Time Strategy (WRONG):**
Example: Capacity 10
- Item A: weight 6, value 30 (ratio 5)
- Item B: weight 5, value 28 (ratio 5.6)
- Item C: weight 5, value 28 (ratio 5.6)

For activity selection with earliest start time greedy fails because it doesn't leave room for optimal selections.

---

## PART 7: ELEMENTS OF GREEDY STRATEGY

### 7.1 Greedy Choice Property

A problem exhibits the greedy choice property if:
- A globally optimal solution can be arrived at by making locally optimal choices
- The choice made doesn't depend on future choices

**Testing for Greedy Choice Property:**

1. **Formalize the choice:** What exactly is "greedy"?
2. **Prove safe:** Show greedy choice is part of some optimal solution
3. **Show optimality:** Greedy choice + optimal subproblem = optimal solution

### 7.2 Optimal Substructure

After a greedy choice, the remaining problem must be of the same form as the original.

**Example (Activity Selection):**
- Choose activity 1 (earliest finish)
- Remaining problem: Select activities compatible with 1
- This is same as original problem on restricted set
- ✓ Has optimal substructure

### 7.3 When Greedy FAILS

**Traveling Salesman Problem (TSP):**
Greedy nearest neighbor does NOT guarantee optimal tour.

**0/1 Knapsack:**
Greedy by value/weight ratio fails because items are discrete.

**Fractional Knapsack (Greedy Works!):**
- Greedy: Take items by highest value/weight ratio
- This DOES give optimal solution

---

## PART 8: FRACTIONAL KNAPSACK PROBLEM

### 8.1 Problem Definition

Given:
- Items with weights wᵢ and values vᵢ
- Knapsack capacity W
- Can take fractional amounts of items

Find: Maximum value with total weight ≤ W

**Difference from 0/1 Knapsack:**
- 0/1: Take whole item or nothing → NP-hard (needs DP)
- Fractional: Take any amount → Greedy works!

**Example:**
```
Items:     1    2    3
Weight:    2    3    5
Value:     10   15   30
Ratio:     5.0  5.0  6.0

Capacity W = 7

Greedy (by ratio):
- Take 5 units of item 3 (ratio 6.0): value = 30, weight = 5
- Take 2 units of item 1 (ratio 5.0): value = 10, weight = 2
- Total: value = 40, weight = 7

This is optimal!
```

### 8.2 Algorithm

```
FRACTIONAL-KNAPSACK(items, W):
    // Sort items by value/weight ratio in decreasing order
    sort items by (value[i] / weight[i]) descending
    
    total_value = 0
    remaining_weight = W
    
    for each item i in sorted order
        if weight[i] <= remaining_weight
            // Take whole item
            total_value += value[i]
            remaining_weight -= weight[i]
        else
            // Take fractional part
            fraction = remaining_weight / weight[i]
            total_value += fraction × value[i]
            remaining_weight = 0
            break
    
    return total_value

Time Complexity: O(n log n) for sorting + O(n) for selection = O(n log n)
Space Complexity: O(n)
```

### 8.3 Proof of Correctness

**Theorem:** Greedy algorithm (highest value/weight ratio first) gives optimal solution.

**Proof:**

Assume for contradiction that greedy is not optimal.

Let G be greedy solution, O be optimal solution that differs from G.

Since they differ, there's a first item where they disagree.
- Greedy includes item j (highest ratio not yet considered)
- Optimal O doesn't include j, or includes less of j

**Case 1:** O doesn't include j at all
- O has space for something else (lower ratio)
- Replace that lower-ratio item with j
- This increases value (higher ratio = better)
- O wasn't optimal. Contradiction.

**Case 2:** O includes j but less than greedy
- Greedy took as much as possible
- O has empty space or lower-ratio item
- Replace that with more of j
- This increases value
- O wasn't optimal. Contradiction.

Therefore, greedy IS optimal. ∎

### 8.4 Why Fractional Works but 0/1 Doesn't

**Fractional:**
- Continuously exchange lower-ratio for higher-ratio items
- Local swaps lead to global optimum
- Greedy property holds

**0/1 Knapsack:**
- Can't make continuous exchanges
- Discrete choices create complications
- Requires trying all subsets (exponential) or DP (polynomial)

---

## PART 9: HUFFMAN CODING

### 9.1 Problem Definition

Huffman Coding creates optimal prefix-free binary codes for a set of characters.

**Prefix-Free Code:** No codeword is a prefix of another
- Example: {0, 10, 11} is prefix-free
- Example: {01, 011} is NOT prefix-free (01 is prefix of 011)

**Goal:** Minimize expected length of encoded message

**Example:**
```
Characters: a, b, c, d
Frequencies: 45%, 13%, 12%, 30%

Fixed-length encoding: 2 bits per character
Expected: 2 bits

Huffman encoding:
a: 0 (1 bit, 45% frequency) → 0.45 bits average
b: 101 (3 bits, 13% frequency) → 0.39 bits average
c: 100 (3 bits, 12% frequency) → 0.36 bits average
d: 11 (2 bits, 30% frequency) → 0.60 bits average

Total expected: 0.45 + 0.39 + 0.36 + 0.60 = 1.80 bits (vs 2.00 fixed)
Savings: 10%
```

### 9.2 The Huffman Algorithm

**Key Idea:** Build binary tree bottom-up
- Frequent characters closer to root (shorter codes)
- Rare characters farther from root (longer codes)

**Algorithm:**

```
HUFFMAN(C):  // C is set of characters with frequencies
    n = |C|
    
    // Create min-heap of single-character trees
    Q = min-heap(C)  // ordered by frequency
    
    for i = 1 to n-1
        // Extract two minimum-frequency nodes
        z = new InternalNode()
        x = EXTRACT-MIN(Q)
        y = EXTRACT-MIN(Q)
        
        // Create new node with sum frequency
        z.freq = x.freq + y.freq
        z.left = x
        z.right = y
        
        // Insert back into heap
        INSERT(Q, z)
    
    // Final node is root
    return EXTRACT-MIN(Q)

Time Complexity: O(n log n)
Why? Build heap: O(n)
     Loop (n-1) times: each iteration does 2 extract-min + 1 insert = O(log n)
     Total: O(n) + (n-1)×O(log n) = O(n log n)

Space Complexity: O(n)
```

### 9.3 Complete Example

**Characters with frequencies:**
```
a: 45
b: 13
c: 12
d: 30
```

**Step 1: Create single-node trees**
```
Q = [c:12, b:13, d:30, a:45]
```

**Step 2: Iteration 1 - Merge c and b**
```
Extract min: c:12, b:13
Create node: cb:25
Insert: Q = [d:30, a:45, cb:25]
Reorganize: Q = [cb:25, d:30, a:45]
```

**Step 3: Iteration 2 - Merge cb and d**
```
Extract min: cb:25, d:30
Create node: cbd:55
Insert: Q = [a:45, cbd:55]
```

**Step 4: Iteration 3 - Merge a and cbd**
```
Extract min: a:45, cbd:55
Create node: root:100
Q = [root:100]
```

**Final Tree:**
```
           root(100)
          /        \
        a(45)    cbd(55)
                /        \
             cb(25)      d(30)
            /      \
          c(12)   b(13)
```

**Resulting Codes:**
```
a: 0        (1 bit)
d: 10       (2 bits)
c: 110      (3 bits)
b: 111      (3 bits)

Proof of optimality:
Expected code length = 45×1 + 30×2 + 12×3 + 13×3
                     = 45 + 60 + 36 + 39
                     = 180 bits per 100 characters
                     = 1.80 bits average
```

### 9.4 Proof of Huffman Optimality

**Theorem:** Huffman's algorithm produces an optimal prefix-free code.

**Proof Outline:**

**Lemma 1:** There exists an optimal code where the two rarest characters have same parent.
- If not, we can swap and reduce average code length
- Contradiction

**Lemma 2:** If we replace two rarest characters with their parent, the reduced problem has optimal solution that extends to optimal solution for original.
- Parent has frequency = sum of children
- Optimal code for reduced problem + encoding children optimally = optimal for original

**Induction:** Apply Lemma 2 recursively until one character remains (trivial).

Therefore, Huffman builds optimal code. ∎

### 9.5 Encoding and Decoding

**Encoding (Trivial):**
```
HUFFMAN-ENCODE(message, codes):
    result = ""
    for each character c in message
        result += codes[c]
    return result

Time: O(message_length)
```

**Decoding (Using Tree):**
```
HUFFMAN-DECODE(encoded_bits, root):
    result = ""
    current = root
    
    for each bit in encoded_bits
        if bit == 0
            current = current.left
        else
            current = current.right
        
        if current is leaf
            result += current.character
            current = root
    
    return result

Time: O(encoded_length)
```

---

## PART 10: COMPARISON - DP vs GREEDY

| Aspect | Dynamic Programming | Greedy |
|--------|-------------------|--------|
| Approach | Bottom-up: Build optimal from subproblems | Top-down: Make locally optimal choice |
| Substructure | Required (optimal substructure) | Required (optimal substructure) |
| Subproblem Choice | Try ALL options, pick best | Choose ONE locally best option |
| Correctness | Always correct if properly formulated | Must prove greedy choice property |
| Time | Usually higher (multiple subproblems) | Usually lower (single choice) |
| Space | Needs DP table | Usually O(n) or less |
| Intuition | "What if we try everything?" | "What's best right now?" |

**When to Use DP:**
- Knapsack (0/1)
- Longest Increasing Subsequence
- Edit Distance
- Matrix Chain Multiplication
- Any problem with overlapping subproblems

**When to Use Greedy:**
- Activity Selection (proven safe)
- Fractional Knapsack (proven safe)
- Huffman Coding (proven optimal)
- Minimum Spanning Tree (Kruskal's/Prim's)
- Dijkstra's Shortest Path

**Hybrid Approaches:**
- Some problems need DP with greedy choices
- Some need greedy for approximation (when optimal is hard)

---

## PART 11: EXAM PRACTICE PROBLEMS

### Problem 1: Matrix Chain Multiplication

Given matrices:
- A₁: 10×30
- A₂: 30×5
- A₃: 5×60
- A₄: 60×8

Solve the matrix chain multiplication problem:
1. Build the cost table m[i,j]
2. Build the split table s[i,j]
3. Write the optimal parenthesization
4. Compute total number of scalar multiplications
5. Write the time and space complexity

---

### Problem 2: LCS Application

Given:
```
X = "ALGORITHM"
Y = "ALTRUISM"
```

1. Construct the LCS table (show all c[i,j] values)
2. Find the length of LCS
3. Trace back and print the actual LCS string
4. How many different LCS of this length exist?

---

### Problem 3: Activity Selection

Given activities:
```
Activity  1   2   3   4   5   6
Start     0   1   3   2   5   4
Finish    4   2   5   6   8   7
```

1. Sort by finish time
2. Apply the greedy algorithm step-by-step
3. Show which activities are selected
4. Prove why your solution is optimal

---

### Problem 4: Fractional Knapsack

```
Items:     1    2    3    4
Weight:    3    4    5    2
Value:     60   90   100  50
Capacity:  10

Knapsack Capacity = 10
```

1. Compute value/weight ratio for each item
2. Sort by ratio
3. Apply greedy algorithm
4. Show what goes in the knapsack (whole items and fractions)
5. Calculate total value

---

### Problem 5: Huffman Coding

Given character frequencies:
```
Character  Frequency
e          32
a          20
b          15
c          15
d          18
```

1. Build the Huffman tree step-by-step
2. Assign binary codes to each character
3. What is the average code length?
4. Show how you would encode "bead"
5. Show how you would decode "0110011001"

---

### Problem 6: Compare Approaches

You need to find the longest common subsequence of two DNA strands.

1. Could you use a greedy algorithm? Why or why not?
2. Why must you use dynamic programming?
3. What are the recurrence relations?
4. Complexity analysis

---

### Problem 7: Counterexample Problem

Why doesn't a greedy "always take the most expensive item first" strategy work for 0/1 knapsack problem?

Give a specific counterexample with:
- Item weights and values
- Knapsack capacity
- Show where greedy fails
- Show optimal solution

---

### Problem 8: Prove Greedy Safety

Prove that the greedy choice property holds for the activity selection problem. Use the "exchange argument" technique:

1. Define the greedy choice clearly
2. Assume there exists an optimal solution with a different first choice
3. Show you can exchange the first choice with the greedy choice
4. Prove the resulting solution is still optimal
5. Conclude by induction

---

## PART 12: EXAM ANSWER FORMAT EXAMPLES

### Example 1: Matrix Chain Multiplication Answer

**Question:** Explain matrix chain multiplication and its solution.

**Answer:**

**Explanation:**
Matrix chain multiplication is an optimization problem where we want to find the most efficient way to compute a product of matrices. Although matrix multiplication is associative (different parenthesizations give same result), the number of scalar multiplications varies dramatically based on how we parenthesize. The goal is to find the parenthesization that minimizes scalar multiplications.

**Recurrence Relation:**
Let m[i,j] be the minimum cost to multiply matrices from i to j.
```
m[i,j] = 0                                    if i = j
m[i,j] = min(m[i,k] + m[k+1,j] + p[i-1]p[k]p[j])  if i < j
         for i ≤ k < j
```

Where p[i-1] × p[i] is the dimension of matrix A_i (rows × columns).

**Algorithm (Pseudocode):**
```
MATRIX-CHAIN-ORDER(p):
    n = length(p) - 1
    for i = 1 to n
        m[i,i] = 0
    for len = 2 to n
        for i = 1 to n - len + 1
            j = i + len - 1
            m[i,j] = ∞
            for k = i to j-1
                cost = m[i,k] + m[k+1,j] + p[i-1]×p[k]×p[j]
                if cost < m[i,j]
                    m[i,j] = cost
                    s[i,j] = k
    return m, s
```

**Complexity Analysis:**
- **Time:** O(n³) - three nested loops with innermost iterating k values
  - Outer loop: n iterations
  - Middle loop: at most n iterations
  - Inner loop: at most n iterations
  - Total: (n)(n)(n)/6 ≈ n³/6 = O(n³)

- **Space:** O(n²) - for m table and s table storage

**Why DP is Needed:**
This problem exhibits optimal substructure (optimal solution uses optimal solutions of subproblems) and overlapping subproblems (same ranges computed multiple times in naive recursion). Therefore, dynamic programming with memoization is ideal.

---

### Example 2: Activity Selection Answer

**Question:** Solve the activity selection problem and prove correctness.

**Answer:**

**Explanation:**
The activity selection problem asks: given activities with start and finish times, select the maximum number of non-overlapping activities. This is solved greedily by always selecting the activity that finishes earliest, as this leaves the most room for remaining activities.

**Algorithm (Pseudocode):**
```
ACTIVITY-SELECTION(s, f):
    n = length(s)
    sort activities by finish time f[i]
    
    A = {1}  // select first activity
    k = 1
    for i = 2 to n
        if s[i] >= f[k]
            A = A ∪ {i}
            k = i
    return A
```

**Complexity:**
- **Time:** O(n log n) - dominated by sorting
- **Space:** O(1) - only storing indices

**Correctness Proof:**

We prove using the exchange argument that greedy always produces an optimal solution.

Let A be the greedy solution with activities a₁, a₂, ..., aₖ (sorted by finish time).
Let O be any optimal solution with activities o₁, o₂, ..., oₘ.

**Base Case:** The first greedy choice a₁ finishes earliest among all activities. Any optimal solution starting with a different activity o₁ can exchange o₁ for a₁ without conflict (since a₁ finishes earlier). The resulting solution remains valid and optimal.

**Inductive Step:** After choosing the first k activities, the problem for remaining activities is identical in structure. Applying the same argument, the greedy choice for position k+1 can replace any other choice in the optimal solution without loss.

**Conclusion:** By induction, greedy solution equals or exceeds any optimal solution. Since optimality is an upper bound, greedy IS optimal. ✓

---

### Example 3: Huffman Coding Answer

**Question:** Explain Huffman coding and code your example.

**Answer:**

**Explanation:**
Huffman coding creates optimal variable-length prefix-free binary codes. Characters that appear more frequently receive shorter codes, reducing overall message length. The algorithm builds a binary tree bottom-up by repeatedly merging the two least-frequent characters.

**Algorithm (Pseudocode):**
```
HUFFMAN(C):
    n = |C|
    Q = min-heap(C)  // heap of character frequencies
    
    for i = 1 to n-1
        z = new Node()
        x = EXTRACT-MIN(Q)
        y = EXTRACT-MIN(Q)
        z.freq = x.freq + y.freq
        z.left = x
        z.right = y
        INSERT(Q, z)
    
    return EXTRACT-MIN(Q)  // root of tree

ENCODE(message, codes):
    return concatenation of codes[c] for each c in message

DECODE(bitstring, root):
    result = ""
    current = root
    for each bit in bitstring
        current = left_child if bit=0 else right_child
        if current is leaf
            result += current.char
            current = root
    return result
```

**Complexity:**
- **Time:** O(n log n) - heap operations inside loop
  - Build heap: O(n)
  - Loop n-1 times, each with 3 heap operations: O(log n)
  - Total: O(n log n)

- **Space:** O(n) - for heap and tree

**Example Execution:**
Characters: a(45), b(13), c(12), d(30)

Step 1: Merge c(12) + b(13) → cb(25)
Step 2: Merge cb(25) + d(30) → cbd(55)
Step 3: Merge a(45) + cbd(55) → root(100)

**Resulting Codes:**
- a: 0
- d: 10
- c: 110
- b: 111

**Why Optimal:**
Huffman's greedy choice (merge two smallest frequencies) is provably optimal. The proof uses exchange argument: if any other pairing existed in optimal solution, we could improve it by using Huffman's pairing, contradicting optimality.

---

## PART 13: QUICK REFERENCE

### Dynamic Programming Patterns

```
Matrix Chain Multiplication:
- Time: O(n³)
- Space: O(n²)
- Key: Optimal split point in interval

Longest Common Subsequence:
- Time: O(m×n)
- Space: O(m×n) [can optimize to O(min(m,n))]
- Key: Match characters or skip one

General DP:
1. Define subproblems clearly
2. Express recurrence relation
3. Build table bottom-up (or memoize top-down)
4. Extract solution from table
```

### Greedy Proof Checklist

```
To prove greedy is optimal:
☐ State the greedy choice precisely
☐ Prove greedy choice is safe (part of some optimal)
☐ Show remaining problem is same type as original
☐ Apply induction or exchange argument
☐ Conclude greedy is optimal
```

### Algorithm Comparison

```
PROBLEM          APPROACH      WHY
Activity Sel.    Greedy        Earliest finish is safe choice
Frac Knapsack    Greedy        Highest ratio is safe choice
0/1 Knapsack     DP            Discrete choices, no greedy property
LCS              DP            Overlapping subproblems
Matrix Chain     DP            Overlapping subproblems
Huffman          Greedy        Merge smallest is optimal
```

### Complexity Cheat Sheet

```
Sorting: O(n log n) [required for greedy]
DP Table: O(m×n) if 2D, O(n³) if 3 nested loops
Tree Building: O(n log n) with heap
Tree Traversal: O(n)
```

---

## PART 14: MEMORY AIDS

**Matrix Chain - Remember MCM:**
- M = Minimize multiplications
- C = Chain of matrices
- M = Many subproblems (DP needed)

**LCS - Remember Three Cases:**
1. Characters match: take both + solve rest
2. Don't match: try skipping first char
3. Don't match: try skipping second char

**Greedy Approach - Remember GAG:**
- G = Greedy choice (locally optimal)
- A = All subproblems solved greedily
- G = Global optimum achieved

**Huffman - Remember HUF:**
- H = Heap of frequencies
- U = Upward building (bottom to top)
- F = Frequent chars near root (short codes)

---

End of Unit II Study Guide
