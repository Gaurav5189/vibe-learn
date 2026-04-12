# DAA Unit III — Complete Study Guide
### MCA | Design and Analysis of Algorithms

---

## TABLE OF CONTENTS
1. [Disjoint Sets](#1-disjoint-sets)
2. [Graph Representations](#2-graph-representations)
3. [BFS & DFS](#3-bfs--dfs)
4. [Minimum Spanning Trees](#4-minimum-spanning-trees-mst)
5. [Single-Source Shortest Paths](#5-single-source-shortest-paths)
6. [All-Pairs Shortest Paths](#6-all-pairs-shortest-paths-floyd-warshall)
7. [Maximum Flow](#7-maximum-flow)
8. [Quick Revision Table](#8-quick-revision-table)

---

## 1. Disjoint Sets

### What is a Disjoint Set (Union-Find)?
A **Disjoint Set** is a data structure that tracks a collection of non-overlapping sets. Each set has a **representative** (root element).

**Three core operations:**
| Operation | Description |
|---|---|
| `MAKE-SET(x)` | Creates a new set with only element x |
| `FIND-SET(x)` | Returns the representative of the set containing x |
| `UNION(x, y)` | Merges the sets containing x and y |

---

### 1.1 Linked-List Representation

Each set is a linked list. The **head** of the list is the representative.

```
Set {A, B, C}:   [A] → [B] → [C]
                  ↑ representative
```

- `MAKE-SET` → O(1)
- `FIND-SET` → O(1) if each node has pointer to head
- `UNION` → O(n) naively (append one list to another)

**Weighted Union Heuristic:** Always append the **shorter** list onto the **longer** one.
- Amortized cost of n operations: **O(n log n)**

---

### 1.2 Disjoint-Set Forests

Each set is a **rooted tree**. Each node points to its **parent**. Root points to itself.

```
    A          D
   / \          \
  B   C          E
```
- `FIND-SET(B)` → follow parent pointers to root → A

**Two key optimizations:**

#### i) Union by Rank
- Attach the tree with **smaller rank** under the tree with **larger rank**.
- Rank = upper bound on height of node.

```
UNION(A, D):  if rank[A] > rank[D] → D becomes child of A
```

#### ii) Path Compression
- During `FIND-SET`, make **every node** on the path point directly to root.

```
Before: B → C → A (root)
After FIND-SET(B): B → A, C → A  (flattened!)
```

**Combined complexity:** O(α(n)) per operation — practically **O(1)**
*(α = inverse Ackermann function, grows extremely slowly)*

> **Exam Tip:** Always mention both optimizations together. The combined amortized cost is O(α(n)).

---

## 2. Graph Representations

A graph G = (V, E) where V = vertices, E = edges.

### 2.1 Adjacency Matrix
- 2D array `A[V][V]`
- `A[i][j] = 1` if edge (i,j) exists, else 0
- Space: **O(V²)**
- Best for **dense graphs**

```
Graph:  1--2, 1--3, 2--3

     1  2  3
  1 [0, 1, 1]
  2 [1, 0, 1]
  3 [1, 1, 0]
```

### 2.2 Adjacency List
- Array of V lists; each list stores neighbors
- Space: **O(V + E)**
- Best for **sparse graphs**

```
1 → [2, 3]
2 → [1, 3]
3 → [1, 2]
```

| | Adjacency Matrix | Adjacency List |
|---|---|---|
| Space | O(V²) | O(V+E) |
| Check edge (u,v) | O(1) | O(degree(u)) |
| List all neighbors | O(V) | O(degree(u)) |

---

## 3. BFS & DFS

### 3.1 Breadth-First Search (BFS)

**Idea:** Explore level by level from a source vertex s. Uses a **Queue**.

**Algorithm:**
```
BFS(G, s):
  color all vertices WHITE, d[v] = ∞, π[v] = NIL
  color[s] = GRAY, d[s] = 0
  Enqueue(Q, s)
  while Q not empty:
      u = Dequeue(Q)
      for each v in Adj[u]:
          if color[v] == WHITE:
              color[v] = GRAY
              d[v] = d[u] + 1
              π[v] = u
              Enqueue(Q, v)
      color[u] = BLACK
```

- **Time Complexity:** O(V + E)
- **Finds:** Shortest path (unweighted), BFS tree

**Example:**
```
Graph: 1-2, 1-3, 2-4, 3-4

BFS from 1:
Queue: [1] → dequeue 1, enqueue 2,3
Queue: [2,3] → dequeue 2, enqueue 4
Queue: [3,4] → dequeue 3 (4 already visited)
Queue: [4] → dequeue 4

Distances: d[1]=0, d[2]=1, d[3]=1, d[4]=2
```

---

### 3.2 Depth-First Search (DFS)

**Idea:** Go as deep as possible before backtracking. Uses **Stack** (or recursion).

**Algorithm:**
```
DFS(G):
  for each vertex u: color[u] = WHITE, π[u] = NIL
  time = 0
  for each vertex u:
      if color[u] == WHITE: DFS-VISIT(u)

DFS-VISIT(u):
  color[u] = GRAY
  time = time + 1; d[u] = time        // discovery time
  for each v in Adj[u]:
      if color[v] == WHITE:
          π[v] = u
          DFS-VISIT(v)
  color[u] = BLACK
  time = time + 1; f[u] = time        // finish time
```

- **Time Complexity:** O(V + E)

**Edge Classifications in DFS:**
| Edge Type | Condition |
|---|---|
| Tree Edge | v is WHITE when (u,v) explored |
| Back Edge | v is GRAY → cycle exists! |
| Forward Edge | v is BLACK, d[u] < d[v] |
| Cross Edge | v is BLACK, d[u] > d[v] |

> **Exam Tip:** Back edges indicate cycles. In a DFS of a **DAG**, no back edges exist.

**Applications of DFS:**
- Topological Sort (finish times in decreasing order)
- Detecting cycles
- Finding Strongly Connected Components (Kosaraju's Algorithm)

---

## 4. Minimum Spanning Trees (MST)

**Definition:** A spanning tree of graph G that connects all vertices with **minimum total edge weight** and no cycles.

**Properties:**
- Has exactly **V-1 edges**
- Unique if all edge weights are distinct

---

### 4.1 Kruskal's Algorithm

**Strategy:** Greedy — Add edges in **increasing weight order**, skip if it forms a cycle.

**Uses Disjoint Set (Union-Find) to detect cycles.**

```
KRUSKAL(G, w):
  A = ∅
  for each vertex v: MAKE-SET(v)
  Sort edges by weight (ascending)
  for each edge (u, v) in sorted order:
      if FIND-SET(u) ≠ FIND-SET(v):    // no cycle
          A = A ∪ {(u,v)}
          UNION(u, v)
  return A
```

**Time Complexity:** O(E log E) or O(E log V)

**Example:**
```
Edges (sorted): (1,2,1), (3,4,2), (2,3,3), (1,4,5)

Step 1: Add (1,2,1) → MST: {1-2}
Step 2: Add (3,4,2) → MST: {1-2, 3-4}
Step 3: Add (2,3,3) → connects components → MST: {1-2, 3-4, 2-3}
Step 4: Skip (1,4,5) → would form cycle

MST weight = 1+2+3 = 6
```

---

### 4.2 Prim's Algorithm

**Strategy:** Greedy — Grow MST one vertex at a time. Always pick the **minimum weight edge** crossing the cut.

**Uses a Min-Priority Queue.**

```
PRIM(G, w, r):
  for each u: key[u] = ∞, π[u] = NIL
  key[r] = 0
  Q = all vertices (min-heap by key)
  while Q not empty:
      u = EXTRACT-MIN(Q)
      for each v in Adj[u]:
          if v ∈ Q and w(u,v) < key[v]:
              π[v] = u
              key[v] = w(u,v)    // decrease-key
```

**Time Complexity:**
- With binary heap: **O(E log V)**
- With Fibonacci heap: O(E + V log V)

**Example (same graph):**
```
Start at vertex 1. key = [0, ∞, ∞, ∞]

Extract 1: update neighbors → key[2]=1, key[4]=5
Extract 2: update neighbors → key[3]=3
Extract 3: update neighbors → key[4]=2 (improved!)
Extract 4

MST edges: 1-2 (1), 2-3 (3), 3-4 (2) → Total = 6 ✓
```

| | Kruskal's | Prim's |
|---|---|---|
| Approach | Edge-based | Vertex-based |
| Data Structure | Union-Find | Priority Queue |
| Best for | Sparse graphs | Dense graphs |
| Complexity | O(E log E) | O(E log V) |

---

## 5. Single-Source Shortest Paths

Find shortest path from **one source vertex** to all others.

**Key concept — Relaxation:**
```
RELAX(u, v, w):
  if d[v] > d[u] + w(u,v):
      d[v] = d[u] + w(u,v)
      π[v] = u
```

---

### 5.1 Bellman-Ford Algorithm

**Handles:** Graphs with **negative weight edges** (detects negative cycles).

```
BELLMAN-FORD(G, w, s):
  INITIALIZE: d[s]=0, d[v]=∞ for all others, π[v]=NIL
  for i = 1 to |V|-1:
      for each edge (u,v) ∈ E:
          RELAX(u, v, w)
  // Check for negative-weight cycles
  for each edge (u,v) ∈ E:
      if d[v] > d[u] + w(u,v):
          return FALSE  // negative cycle exists!
  return TRUE
```

**Time Complexity:** **O(VE)**

**Example:**
```
Vertices: {s, A, B, C}
Edges: s→A(6), s→B(7), A→B(8), A→C(5), B→C(-3), C→A(-2)

After 1st iteration: d[A]=6, d[B]=7, d[C]=4 (via B→C: 7-3=4)
After 2nd iteration: d[A]=2 (via C→A: 4-2=2), d[B]=10...
...continues until stable
```

> **Exam Tip:** Bellman-Ford runs **V-1** relaxation passes because the longest simple path has at most V-1 edges.

---

### 5.2 Dijkstra's Algorithm

**Handles:** Graphs with **non-negative weights only** (faster than Bellman-Ford).

```
DIJKSTRA(G, w, s):
  INITIALIZE: d[s]=0, d[v]=∞, π[v]=NIL
  S = ∅ (set of finalized vertices)
  Q = all vertices (min-heap by d)
  while Q not empty:
      u = EXTRACT-MIN(Q)
      S = S ∪ {u}
      for each v in Adj[u]:
          RELAX(u, v, w)   // updates key in Q if d[v] improves
```

**Time Complexity:**
- With binary heap: **O((V + E) log V)**
- With Fibonacci heap: O(V log V + E)

**Example:**
```
Graph: s-A(10), s-C(3), A-B(2), C-A(4), C-B(8), B-D(7)

Extract s(0):  d[A]=10, d[C]=3
Extract C(3):  d[A]=min(10,7)=7, d[B]=11
Extract A(7):  d[B]=min(11,9)=9
Extract B(9):  d[D]=16
Extract D(16): done

Shortest paths from s: A=7, B=9, C=3, D=16
```

| | Bellman-Ford | Dijkstra |
|---|---|---|
| Negative weights | ✅ Yes | ❌ No |
| Negative cycles | Detects them | Cannot handle |
| Complexity | O(VE) | O((V+E) log V) |
| Approach | Relax all edges V-1 times | Greedy, relaxes once per vertex |

---

## 6. All-Pairs Shortest Paths: Floyd-Warshall

**Goal:** Find shortest paths between **all pairs** of vertices.

**Idea:** Dynamic Programming — consider each vertex k as an **intermediate vertex**.

```
d[i][j][k] = shortest path from i to j using only vertices {1, 2, ..., k} as intermediates
```

**Recurrence:**
```
d[i][j][0] = w(i,j)         // direct edge weight (∞ if no edge)
d[i][j][k] = min( d[i][j][k-1],  d[i][k][k-1] + d[k][j][k-1] )
                 (skip k)          (go through k)
```

**Algorithm:**
```
FLOYD-WARSHALL(W):
  n = rows of W
  D = W   // initialize distance matrix
  for k = 1 to n:
      for i = 1 to n:
          for j = 1 to n:
              D[i][j] = min(D[i][j], D[i][k] + D[k][j])
  return D
```

**Time Complexity:** **O(V³)**
**Space:** O(V²)

**Example:**
```
    1    2    3
1 [ 0,   3,   8 ]
2 [ ∞,   0,  -4 ]
3 [ 2,   ∞,   0 ]

After k=1: D[2][2] = min(0, ∞+3) = 0, D[3][2] = min(∞, 2+3) = 5
After k=2: D[1][3] = min(8, 3+(-4)) = -1
After k=3: D[2][1] = min(∞, -4+2) = -2

Final D:
    1    2    3
1 [ 0,   3,  -1 ]
2 [-2,   0,  -4 ]
3 [ 2,   5,   0 ]
```

> **Detecting Negative Cycles:** After running Floyd-Warshall, if any `D[i][i] < 0`, a negative cycle exists.

---

## 7. Maximum Flow

### 7.1 Flow Networks

A directed graph G = (V, E) where:
- Each edge (u,v) has **capacity** c(u,v) ≥ 0
- A **source** s and a **sink** t
- **Flow** f(u,v): amount of material on edge (u,v)

**Flow constraints:**
1. **Capacity constraint:** 0 ≤ f(u,v) ≤ c(u,v)
2. **Flow conservation:** For all v ≠ s,t: inflow = outflow

**Value of flow:** |f| = total flow leaving s = total flow entering t

---

### 7.2 Ford-Fulkerson Method

**Key Concepts:**

**Residual Network Gf:**
- For each edge (u,v) with flow f and capacity c:
  - Forward edge: residual capacity = c(u,v) - f(u,v)
  - Backward edge: residual capacity = f(u,v) *(allows "undoing" flow)*

**Augmenting Path:** Any path from s to t in the **residual network**.

**Algorithm:**
```
FORD-FULKERSON(G, s, t):
  for each edge (u,v): f(u,v) = 0
  while ∃ augmenting path p in residual network Gf:
      cf(p) = min residual capacity on path p
      for each edge (u,v) in p:
          f(u,v) += cf(p)     // increase forward flow
          f(v,u) -= cf(p)     // decrease backward flow
  return f
```

**Time Complexity:** **O(E · |f*|)** where |f*| = max flow value
*(If BFS is used to find augmenting path → Edmonds-Karp: O(VE²))*

**Example:**
```
Network:  s →(10)→ A →(10)→ t
          s →(10)→ B →(10)→ t
          A →(1)→  B

Step 1: Path s→A→t, flow=10  (send 10)
Step 2: Path s→B→t, flow=10  (send 10)
Max flow = 20
```

---

### 7.3 Max-Flow Min-Cut Theorem

> **The maximum flow in a network equals the capacity of the minimum cut.**

**Cut (S, T):** A partition of V into S (containing s) and T (containing t).
**Capacity of cut:** Sum of capacities of edges going from S to T.

- Ford-Fulkerson **terminates** when no augmenting path exists
- At termination, the **min-cut** is found simultaneously

> **Exam Tip:** To find the min-cut after Ford-Fulkerson: From s, do BFS/DFS in residual graph. S = reachable vertices, T = rest. Min-cut edges = original edges from S to T.

---

## 8. Quick Revision Table

| Algorithm | Problem | Technique | Time Complexity |
|---|---|---|---|
| **Kruskal** | MST | Greedy + Union-Find | O(E log E) |
| **Prim** | MST | Greedy + Priority Queue | O(E log V) |
| **Bellman-Ford** | SSSP (neg weights) | DP / Relaxation | O(VE) |
| **Dijkstra** | SSSP (non-neg weights) | Greedy + PQ | O((V+E) log V) |
| **Floyd-Warshall** | All-Pairs SP | DP | O(V³) |
| **Ford-Fulkerson** | Max Flow | Augmenting Paths | O(E·|f*|) |
| **BFS** | Unweighted SP, Level traversal | Queue | O(V+E) |
| **DFS** | Cycle detection, Topo sort | Stack/Recursion | O(V+E) |
| **Union-Find** | Disjoint Sets | Path compression + rank | O(α(n)) ≈ O(1) |

---

## Key Exam Points to Remember

1. **Union-Find:** Path compression + union by rank together give O(α(n)) — always mention BOTH.
2. **BFS vs DFS:** BFS → shortest path (unweighted); DFS → cycle detection, topological sort.
3. **Kruskal vs Prim:** Kruskal sorts edges first; Prim grows from a source. Both give correct MST.
4. **Bellman-Ford:** Does V-1 iterations. Can detect negative cycles. Works on negative weights.
5. **Dijkstra:** Fails on negative weights. Greedy, uses priority queue.
6. **Floyd-Warshall:** Triple nested loop. O(V³). Check diagonal for negative cycles.
7. **Ford-Fulkerson:** Augments flow along s-t paths in residual graph until none exist. Max-flow = Min-cut.
8. **Residual Graph:** Always has both forward (remaining capacity) and backward (cancel flow) edges.

---

*End of Unit III — Good Luck! 🎯*
