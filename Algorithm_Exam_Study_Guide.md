# Algorithm Design & Analysis - Unit I Study Guide
## Exam Format: Explanation + Pseudocode + Time Complexity Analysis

---

## PART 1: FOUNDATION TOPICS

### 1.1 Introduction to Algorithm Design & Analysis

**What is an Algorithm?**
A step-by-step procedure to solve a problem with specific inputs and desired outputs.

**Why Analyze Algorithms?**
- Compare different solutions
- Predict performance on large inputs
- Make informed design choices

**Example: Linear Search vs Binary Search**
- Same problem (find element)
- Different time complexities: O(n) vs O(log n)
- Analysis helps us choose wisely

---

## PART 2: GROWTH OF FUNCTIONS & BIG-O NOTATION

### Key Complexity Classes (You Know This, Quick Review)

```
O(1)        Constant time
O(log n)    Logarithmic
O(n)        Linear
O(n log n)  Linearithmic
O(n²)       Quadratic
O(2ⁿ)       Exponential
O(n!)       Factorial
```

**Intuition:** Big-O describes how an algorithm scales as input grows.

---

## PART 3: RECURRENCES (YOUR WEAK AREA - LET'S MASTER THIS!)

### What is a Recurrence Relation?

A recurrence relation expresses the time complexity of a recursive algorithm in mathematical form.

**Example 1: Binary Search**
```
T(n) = T(n/2) + O(1)
```
This says: "Time for binary search on n items = time for half the size + constant work"

**Example 2: Merge Sort**
```
T(n) = 2T(n/2) + O(n)
```
This says: "Time for merge sort = 2 recursive calls on halves + O(n) to merge"

### Why Write Recurrences?

When you have a recursive algorithm, you can't just call it O(n) or O(n²). You need to write a recurrence that captures the recursive structure, then solve it.

---

## PART 4: THREE METHODS TO SOLVE RECURRENCES

### Method 1: Substitution (Guess and Verify)

**Steps:**
1. **Guess** a solution form: T(n) = O(f(n))
2. **Substitute** it into the recurrence
3. **Verify** using mathematical induction

**Example: T(n) = 2T(n/2) + n**

**Step 1 - Guess:** This looks like it might be O(n log n). Let's guess T(n) = cn log n

**Step 2 - Substitute:**
```
T(n) = 2T(n/2) + n
     = 2(c·(n/2)·log(n/2)) + n
     = cn·log(n/2) + n
     = cn·(log n - 1) + n
     = cn log n - cn + n
     = cn log n + n(1 - c)
```

**Step 3 - Verify:**
If we choose c = 1, then:
T(n) = n log n + n(1-1) = n log n ✓

So T(n) = O(n log n)

**When to use:** Small recurrences, or when you have a good guess about the answer.

---

### Method 2: Recursion Tree

**Concept:** Draw a tree showing all recursive calls. Each level's cost adds up, then sum across all levels.

**Example: T(n) = 2T(n/2) + n**

```
                n                    Cost: n
              /   \
           n/2     n/2               Cost: n (2 nodes × n/2)
          /  \    /  \
        n/4  n/4 n/4 n/4             Cost: n (4 nodes × n/4)
        ...
        
Tree depth: log₂(n)
Cost per level: n
Total cost: n × log₂(n) = O(n log n)
```

**Key Insight:**
- Count how many levels in the tree
- What's the cost at each level?
- Sum across all levels

**When to use:** When you can visualize the recursive structure clearly.

---

### Method 3: Master Method (MOST POWERFUL!)

**When to use:** Recurrences of the form **T(n) = aT(n/b) + f(n)**

Where:
- **a** = number of subproblems
- **b** = factor by which problem size reduces
- **f(n)** = cost of dividing and combining

**The Three Cases:**

**CASE 1:** If f(n) = O(n^(log_b(a) - ε)) for some ε > 0
```
→ T(n) = Θ(n^(log_b(a)))
```
(The recursive work dominates)

**CASE 2:** If f(n) = Θ(n^(log_b(a)))
```
→ T(n) = Θ(n^(log_b(a)) × log n)
```
(Recursive work and division/merge are equal)

**CASE 3:** If f(n) = Ω(n^(log_b(a) + ε)) for some ε > 0, AND f(n) satisfies regularity
```
→ T(n) = Θ(f(n))
```
(The division/merge work dominates)

**Quick Decision Tree:**

First, calculate: **log_b(a)**

Then compare f(n) with n^(log_b(a))

---

### Master Method Examples

**Example 1: Merge Sort**
```
T(n) = 2T(n/2) + n

a = 2, b = 2, f(n) = n
log_b(a) = log₂(2) = 1

Compare f(n) = n with n^1 = n
→ They're equal! This is CASE 2

T(n) = Θ(n × log n) = O(n log n) ✓
```

**Example 2: Binary Search**
```
T(n) = T(n/2) + 1

a = 1, b = 2, f(n) = 1
log_b(a) = log₂(1) = 0

Compare f(n) = 1 with n^0 = 1
→ They're equal! This is CASE 2

T(n) = Θ(1 × log n) = O(log n) ✓
```

**Example 3: Strassen's Matrix Multiplication**
```
T(n) = 7T(n/2) + O(n²)

a = 7, b = 2, f(n) = n²
log_b(a) = log₂(7) ≈ 2.807

Compare f(n) = n² with n^2.807
→ n² < n^2.807, so f(n) is much smaller. This is CASE 1

T(n) = Θ(n^2.807) ✓
```

---

## PART 5: SORTING ALGORITHMS

### Quick Refresher: Worst Case Analysis

**Merge Sort**
```
MERGE-SORT(A, p, r)
    if p < r
        q = ⌊(p + r) / 2⌋
        MERGE-SORT(A, p, q)
        MERGE-SORT(A, q+1, r)
        MERGE(A, p, q, r)

Recurrence: T(n) = 2T(n/2) + Θ(n)
Solution: T(n) = Θ(n log n)
Space: Θ(n)
```

**Quick Sort**
```
QUICK-SORT(A, p, r)
    if p < r
        q = PARTITION(A, p, r)
        QUICK-SORT(A, p, q-1)
        QUICK-SORT(A, q+1, r)

Worst Case Recurrence: T(n) = T(n-1) + T(0) + Θ(n) = Θ(n²)
Average Case: T(n) = 2T(n/2) + Θ(n) = Θ(n log n)
Space: Θ(log n) [for recursion stack]
```

**Binary Search**
```
BINARY-SEARCH(A, x, low, high)
    if low > high
        return NOT FOUND
    mid = ⌊(low + high) / 2⌋
    if A[mid] = x
        return mid
    else if A[mid] > x
        return BINARY-SEARCH(A, x, low, mid-1)
    else
        return BINARY-SEARCH(A, x, mid+1, high)

Recurrence: T(n) = T(n/2) + O(1)
Solution: T(n) = O(log n)
```

---

## PART 6: HEAP SORT (YOUR SECOND WEAK AREA)

### 6.1 What is a Heap?

A **binary heap** is a complete binary tree where:
- **Min-heap:** parent ≤ children (parent smallest)
- **Max-heap:** parent ≥ children (parent largest)

**Why Heaps Matter:**
- Efficient priority queue implementation
- Foundation for heap sort
- Used in Dijkstra's algorithm, Prim's algorithm

### 6.2 Heap as Array

Heaps are stored in an array. For element at index i:
- Left child: index 2i
- Right child: index 2i + 1
- Parent: index ⌊i/2⌋

```
        1(10)
       /     \
     2(9)    3(7)
    /  \    /
  4(4) 5(3) 6(2)

Array: [10, 9, 7, 4, 3, 2]
```

### 6.3 Key Heap Operations

**HEAPIFY (Fix Down)**
```
MAX-HEAPIFY(A, i, heap_size)
    l = 2*i                          // left child
    r = 2*i + 1                      // right child
    largest = i
    
    if l ≤ heap_size and A[l] > A[i]
        largest = l
    if r ≤ heap_size and A[r] > A[largest]
        largest = r
    
    if largest ≠ i
        exchange A[i] with A[largest]
        MAX-HEAPIFY(A, largest, heap_size)

Time Complexity: O(log n)
Why? Tree height is log n, and we go down one path
```

**BUILD-HEAP (Create Heap from Array)**
```
BUILD-MAX-HEAP(A)
    heap_size = length(A)
    for i = ⌊length(A)/2⌋ down to 1
        MAX-HEAPIFY(A, i, heap_size)

Time Complexity: O(n)
Why? Not all nodes do O(log n) work. Careful analysis:
- Nodes at height h do O(h) work
- There are n/2^(h+1) nodes at height h
- Sum: Σ (n/2^(h+1)) × h = O(n)
```

**HEAP-EXTRACT-MAX**
```
HEAP-EXTRACT-MAX(A, heap_size)
    max = A[1]
    A[1] = A[heap_size]
    heap_size = heap_size - 1
    MAX-HEAPIFY(A, 1, heap_size)
    return max

Time Complexity: O(log n)
Why? One call to MAX-HEAPIFY
```

**HEAP-INSERT**
```
MAX-HEAP-INSERT(A, key, heap_size)
    heap_size = heap_size + 1
    i = heap_size
    while i > 1 and A[⌊i/2⌋] < key
        A[i] = A[⌊i/2⌋]
        i = ⌊i/2⌋
    A[i] = key

Time Complexity: O(log n)
Why? We bubble up, going one level per step (at most log n levels)
```

### 6.4 Heap Sort Algorithm

```
HEAP-SORT(A)
    BUILD-MAX-HEAP(A)               // O(n)
    heap_size = length(A)
    
    for i = length(A) down to 2
        exchange A[1] with A[i]      // Extract max to end
        heap_size = heap_size - 1
        MAX-HEAPIFY(A, 1, heap_size) // O(log n)
    
    return A

Time Complexity: O(n log n)
Why?
- BUILD-HEAP: O(n)
- Loop runs (n-1) times
- Each iteration: one exchange + one HEAPIFY = O(log n)
- Total: O(n) + (n-1)×O(log n) = O(n log n)

Space Complexity: O(1) [in-place sorting]
```

**Example: Heap Sort on [4, 10, 3, 5, 1]**

Step 1: BUILD-HEAP → [10, 5, 3, 4, 1]
Step 2: Extract 10 → [1, 5, 3, 4, | 10] and heapify [5, 4, 3, 1]
Step 3: Extract 5 → [4, 1, 3, | 5, 10] and heapify [4, 1, 3]
Step 4: Extract 4 → [3, 1, | 4, 5, 10] and heapify [3, 1]
Step 5: Extract 3 → [1, | 3, 4, 5, 10]
Step 6: Extract 1 → [1, 3, 4, 5, 10]

### 6.5 Priority Queues

A priority queue is an abstract data type supporting:
- **INSERT(S, x)** - Add element x, O(log n)
- **EXTRACT-MAX(S)** - Remove and return max, O(log n)
- **FIND-MAX(S)** - Return max without removing, O(1)

Heaps provide an efficient implementation with logarithmic insert and extract.

---

## PART 7: LOWER BOUNDS FOR SORTING

### Comparison-Based Sorting Lower Bound

**Key Insight:** Any comparison-based sorting algorithm needs Ω(n log n) comparisons in the worst case.

**Proof Idea (Decision Tree):**
1. Any comparison algorithm can be represented as a decision tree
2. Each comparison is a binary choice (< or ≥)
3. We need to distinguish n! possible orderings
4. A binary tree with n! leaves needs at least log₂(n!) levels
5. log₂(n!) = Ω(n log n)

**Therefore:** No comparison-based sorting can do better than O(n log n).

**Algorithms That Achieve This Bound:**
- Merge Sort: Θ(n log n)
- Heap Sort: Θ(n log n)
- Quick Sort (average): Θ(n log n)

**Algorithms That Don't (and why):**
- Quick Sort (worst): Θ(n²) - poor pivot choice
- Bubble Sort: Θ(n²) - inefficient comparisons
- Insertion Sort: Θ(n²) - quadratic work

**Non-Comparison Sorts (Can Beat O(n log n)):**
- Counting Sort: O(n + k) where k is range
- Radix Sort: O(d(n + k)) where d is number of digits
- Bucket Sort: O(n + k) average case

---

## PART 8: EXAM PRACTICE PROBLEMS

### Problem 1: Analyze This Recursive Function

```
function FIBONACCI-SORT(arr, n):
    if n ≤ 1
        return
    pivot = arr[n/2]
    left = []
    right = []
    for i = 1 to n
        if arr[i] < pivot
            left.append(arr[i])
        else
            right.append(arr[i])
    FIBONACCI-SORT(left, left.length)
    FIBONACCI-SORT(right, right.length)
    arr = left + [pivot] + right

Write:
1. The recurrence relation
2. Worst case time complexity (with justification)
3. Average case time complexity
4. Space complexity
```

**Solution Template (You'll fill this in):**

Recurrence Relation:
```
T(n) = T(k) + T(n-k-1) + Θ(n)

where k is the number of elements < pivot
Worst case: k = 0 or k = n-1
→ T(n) = T(n-1) + Θ(n)
```

Worst Case Solution:
```
T(n) = T(n-1) + n
     = T(n-2) + (n-1) + n
     = T(1) + 2 + 3 + ... + n
     = Σ(i=1 to n) i
     = Θ(n²)
```

---

### Problem 2: Master Method Practice

For each recurrence, identify a, b, f(n), compute log_b(a), determine the case, and solve:

**A)** T(n) = 4T(n/2) + n
**B)** T(n) = T(n/2) + n²
**C)** T(n) = 16T(n/4) + n²
**D)** T(n) = 2T(n/2) + n log n

*Solutions provided in next section*

---

### Problem 3: Heap Operations

Given: [15, 10, 20, 8, 2, 16, 25]

1. Draw the heap structure
2. Show the array representation
3. Extract the maximum element
4. Show the heap after extraction
5. Insert 18
6. Show final heap

---

### Problem 4: Heapify Trace

Max-heapify the array [1, 10, 5, 3, 2, 4] starting at index 1.
Show each step as the algorithm progresses.

---

### Problem 5: Comparison-Based Sorting Lower Bound

Explain why no comparison-based sorting algorithm can have better than O(n log n) worst-case time complexity. Use decision tree argument.

---

## EXAM ANSWER FORMAT (CRITICAL!)

When answering exam questions, follow this structure:

### Format Example:

**Question:** Analyze the time complexity of merge sort.

**Answer:**

**Explanation:**
Merge sort divides the array into halves recursively until single elements remain. Then it merges them back together in sorted order. The merge operation takes O(n) time to combine two sorted subarrays.

**Recurrence Relation:**
T(n) = 2T(n/2) + Θ(n)

**Algorithm (Pseudocode):**
```
MERGE-SORT(A, p, r)
    if p < r
        q = ⌊(p + r) / 2⌋
        MERGE-SORT(A, p, q)
        MERGE-SORT(A, q+1, r)
        MERGE(A, p, q, r)  // Merges in O(n) time
```

**Solution Method:**
Using the Master Method:
- a = 2, b = 2, f(n) = n
- log_b(a) = log₂(2) = 1
- f(n) = n = n¹ = n^(log_b(a))
- This is CASE 2: T(n) = Θ(n^(log_b(a)) × log n) = Θ(n log n)

**Time Complexity:** O(n log n) ✓
**Space Complexity:** O(n) (requires auxiliary arrays)

---

## QUICK REFERENCE: SOLVING ANY RECURRENCE

```
Step 1: Identify the recurrence form
        T(n) = aT(n/b) + f(n)?  OR  General form?

Step 2: Try Master Method if it fits
        - Calculate log_b(a)
        - Compare f(n) with n^(log_b(a))
        - Determine case 1, 2, or 3
        - Apply formula

Step 3: If Master Method doesn't work, use Recursion Tree
        - Draw tree of recursive calls
        - Calculate cost per level
        - Count levels
        - Sum across levels

Step 4: If still stuck, use Substitution Method
        - Guess the form: O(f(n))
        - Substitute into recurrence
        - Verify by algebra or induction

Step 5: Verify your answer makes sense
        - Does it match the intuition?
        - Is it consistent with examples?
```

---

## WHAT TO MEMORIZE

**Recurrence Solutions:**
- T(n) = T(n/2) + 1 → O(log n)
- T(n) = T(n-1) + 1 → O(n)
- T(n) = 2T(n/2) + n → O(n log n)
- T(n) = T(n-1) + n → O(n²)
- T(n) = 2T(n/2) + 1 → O(n)

**Sorting Algorithms:**
| Algorithm | Best | Average | Worst | Space |
|-----------|------|---------|-------|-------|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) |
| Binary Search | O(log n) | O(log n) | O(log n) | O(1) |

**Heap Operations:**
- BUILD-HEAP: O(n)
- INSERT: O(log n)
- EXTRACT-MAX: O(log n)
- HEAPIFY: O(log n)

---

Now answer these questions to help me understand where to focus our discussion:

1. Of the diagnostic questions I asked earlier, can you attempt them now?
2. Which topic feels most unclear still?
3. Ready to work through practice problems together?
