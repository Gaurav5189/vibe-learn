# Artificial Intelligence (MCC-2.3.4) — Unit I Notes
**PG MCA, Ravenshaw University (CBCS)** | Ref: Rich & Knight (Ch. 1.1, 2, 3); Patterson; Russell & Norvig

**Unit I syllabus:** Introduction to AI · Application areas of AI · State-Space Search (Production system design, Production system characteristics) · Blind search (DFS, BFS) · Heuristic search (Hill Climbing, Best First Search, Branch and Bound, A\*, AO\*)

---

## 1. Introduction to AI

**Definition:** AI is the study of how to make computers do things that, at the moment, people do better (Rich & Knight). It is the science of building machines that exhibit intelligent behaviour: reasoning, learning, perception, problem solving, language understanding.

### Four views of AI (Russell & Norvig)
| | Human-like | Rational |
|---|---|---|
| **Thinking** | Thinking humanly (cognitive modelling) | Thinking rationally (laws of thought, logic) |
| **Acting** | Acting humanly (Turing Test) | Acting rationally (rational agent) |

- **Turing Test (1950):** A machine passes if a human interrogator cannot tell whether responses come from a machine or a person.
- **Rational agent:** Acts to achieve the best expected outcome.

### Foundations of AI
Philosophy (logic, reasoning), Mathematics (logic, probability, computation), Economics (decision theory), Neuroscience, Psychology, Computer Engineering, Control theory, Linguistics.

### Brief history
| Period | Event |
|---|---|
| 1943 | McCulloch & Pitts: artificial neuron model |
| 1950 | Turing: "Computing Machinery and Intelligence" |
| 1956 | Dartmouth Conference: term "AI" coined (John McCarthy) |
| 1960s–70s | General Problem Solver, LISP, early expert systems (DENDRAL, MYCIN) |
| 1980s | Expert-system boom; neural nets revival (backpropagation) |
| 1990s–2000s | Intelligent agents, probabilistic methods, machine learning |
| 2010s+ | Deep learning, big data, generative AI |

### AI problem and techniques
- AI problems often are hard, need much **knowledge**, and cannot be solved by fixed algorithms.
- **AI technique:** A method that exploits knowledge, which should be: (1) capturing generalisations, (2) understandable by people who provide it, (3) easily modifiable to correct errors, (4) usable even if not perfectly accurate, (5) helpful in narrowing the range of possibilities.

---

## 2. Application Areas of AI

| Area | Description / Examples |
|---|---|
| **Expert Systems** | Emulate human experts: MYCIN (medical diagnosis), medical/financial advisory |
| **Natural Language Processing** | Machine translation, chatbots, speech-to-text, sentiment analysis |
| **Robotics** | Industrial robots, autonomous vehicles, drones |
| **Computer Vision / Perception** | Face recognition, medical imaging, object detection |
| **Game Playing** | Chess, Go, board games (Deep Blue, AlphaGo) |
| **Machine Learning / Data Mining** | Prediction, recommendation systems, fraud detection |
| **Theorem Proving / Automated Reasoning** | Proving mathematical theorems, program verification |
| **Planning & Scheduling** | Logistics, timetabling, resource allocation |
| **Healthcare** | Diagnosis support, drug discovery |
| **Finance, Agriculture, Education, Security** | Trading, crop monitoring, tutoring systems, intrusion detection |

---

## 3. State-Space Search

### 3.1 Basic concepts
**State space:** The set of all possible configurations (states) of a problem, together with the operators (rules) that move from one state to another.

A problem is defined by:
1. **Initial state**: where the search starts.
2. **Operators / rules** (successor function): actions that transform a state.
3. **Goal test**: condition(s) that identify a goal state.
4. **Path cost**: numeric cost of a sequence of actions.

**Solution** = a path from initial state to a goal state. **Optimal solution** = the lowest-cost path.

**Steps to formulate a problem as state-space search:**
1. Define the state space (including start and goal states).
2. Specify the rules (legal moves).
3. Choose a control strategy (search technique).

**Example: Water Jug Problem** (4 L and 3 L jugs, get exactly 2 L in the 4 L jug)
- State: (x, y) = water in 4 L and 3 L jugs; start (0, 0); goal (2, n).
- Sample rules: (x, y) → (4, y) if x < 4 [fill 4 L jug]; (x, y) → (0, y) [empty 4 L jug]; pour 3 L into 4 L until full, etc.
- One solution: (0,0) → (0,3) → (3,0) → (3,3) → (4,2) → (0,2) → (2,0).

### 3.2 Production System Design

**Production system:** A framework for problem solving, consisting of a set of rules, a database, and a control strategy.

**Components:**
1. **Set of production rules**: of the form `IF condition THEN action` (condition-action pairs). Rules operate on the database.
2. **Global database (working memory)**: holds the current state/facts of the problem.
3. **Control strategy**: decides *which applicable rule to apply next*; resolves conflicts when several rules match. Should (a) cause motion (progress toward the goal) and (b) be systematic (no state visited needlessly/indefinitely).
4. **Rule applier (interpreter)**: executes the cycle: *match → select (conflict resolution) → apply (fire)*, repeated until the goal is reached.

**Advantages:** modular (rules added/removed independently), natural representation of knowledge, easy to understand, separation of knowledge and control.
**Disadvantages:** inefficient with many rules, no learning (does not store solutions for reuse), rule interactions hard to track.

**Control strategies — two directions of reasoning:**
- **Forward chaining** (data-driven): facts → conclusions.
- **Backward chaining** (goal-driven): goal → required facts.

### 3.3 Production System Characteristics

First, classify the **problem** (Rich & Knight asks these questions):
1. Is the problem **decomposable** into independent subproblems?
2. Can solution steps be **ignored or undone**? (ignorable: theorem proving; recoverable: 8-puzzle; irrecoverable: chess)
3. Is the **universe predictable**? (certain vs. uncertain outcome)
4. Is a **good solution absolute or relative**? (any path vs. best path)
5. Is the solution a **state or a path**?
6. What is the **role of knowledge**?
7. Does the task need **interaction with a person**?

Then, classify the **production system**:

| Type | Definition |
|---|---|
| **Monotonic** | Applying a rule never prevents the later application of another rule that could have been applied earlier. |
| **Non-monotonic** | Applying a rule *may* prevent the later application of another rule that was applicable earlier. |
| **Partially commutative** | If a sequence of rules transforms state X to state Y, then any allowable permutation of those rules also transforms X to Y. |
| **Commutative** | Both monotonic **and** partially commutative. |

| | Monotonic | Non-monotonic |
|---|---|---|
| **Partially commutative** | Theorem proving | Robot navigation, 8-puzzle |
| **Not partially commutative** | Chemical synthesis | Bridge/card games, chess |

- **Partially commutative + monotonic** (commutative) systems: no need to backtrack, since order of rules doesn't matter; ideal for **ignorable** problems.
- **Non-monotonic, not partially commutative**: needed for **irreversible** problems; require careful backtracking.

---

## 4. Search Techniques

**Search strategies** are compared on four criteria:
- **Completeness**: guaranteed to find a solution if one exists?
- **Optimality**: finds the least-cost solution?
- **Time complexity**: number of nodes generated.
- **Space complexity**: maximum nodes stored in memory.

(*b* = branching factor, *d* = depth of shallowest goal, *m* = max depth)

**Two broad classes:**
- **Blind (uninformed)** search: no domain knowledge beyond the problem definition.
- **Heuristic (informed)** search: uses problem-specific knowledge (heuristic function) to guide the search.

---

## 5. Blind Search Techniques

### 5.1 Breadth First Search (BFS)
**Idea:** Expand the shallowest unexpanded node first; explore level by level.
**Data structure:** **Queue (FIFO)**.

**Algorithm:**
1. Put start node in OPEN queue.
2. If OPEN is empty → fail.
3. Remove first node *n* from OPEN; if *n* is the goal → success.
4. Expand *n*; add its children to the **end** of OPEN. Go to step 2.

| Property | Value |
|---|---|
| Complete | Yes (if *b* is finite) |
| Optimal | Yes, if all step costs are equal |
| Time | O(b^d) |
| Space | O(b^d), a major drawback |

**Pros:** finds shortest path. **Cons:** huge memory requirement.

### 5.2 Depth First Search (DFS)
**Idea:** Expand the deepest unexpanded node first; go down a path until a dead end, then backtrack.
**Data structure:** **Stack (LIFO)** or recursion.

**Algorithm:**
1. Put start node on the stack.
2. If stack is empty → fail.
3. Pop top node *n*; if *n* is goal → success.
4. Push its children on **top** of the stack. Go to step 2.

| Property | Value |
|---|---|
| Complete | No (may loop in infinite/cyclic spaces) |
| Optimal | No |
| Time | O(b^m) |
| Space | O(b·m), linear, a major advantage |

**Pros:** low memory; may find a solution quickly if it lies deep. **Cons:** can get trapped in infinite paths; not optimal.

### BFS vs DFS
| | BFS | DFS |
|---|---|---|
| Structure | Queue | Stack |
| Memory | Very high | Low |
| Shortest path | Guaranteed (uniform cost) | Not guaranteed |
| Completeness | Yes | No (infinite depth) |
| Best when | Solution is shallow | Many solutions, deep search space |

---

## 6. Heuristic Search Techniques

**Heuristic:** A rule of thumb/estimate that guides search toward the goal, trading guaranteed optimality for speed.
**Heuristic function h(n):** Estimated cost from node *n* to the nearest goal.
**Evaluation function f(n):** Used to rank nodes for expansion.

### 6.1 Hill Climbing
**Idea:** A local search that always moves to a neighbouring state with a **better** heuristic value; stops when no neighbour is better. No backtracking, no memory of earlier states (like climbing a hill in fog).

**Variants:**
| Variant | Behaviour |
|---|---|
| **Simple hill climbing** | Moves to the *first* neighbour that is better than the current state. |
| **Steepest-ascent hill climbing** | Examines *all* neighbours and moves to the *best* one. |
| **Stochastic hill climbing** | Picks a random better neighbour. |

**Algorithm (simple):**
1. Evaluate the initial state; if it is the goal → stop.
2. Loop: select an operator, generate a new state, evaluate it.
3. If better than the current state → make it the current state; if goal → stop.
4. If no better state can be found → stop.

**Problems:**
- **Local maximum**: a peak higher than its neighbours but lower than the global maximum; search gets stuck.
- **Plateau**: flat area where neighbours have equal values; no direction to move.
- **Ridge**: narrow ascending path; single-step moves cannot follow it.

**Remedies:** random restart, backtracking, simulated annealing, making bigger jumps (apply two rules before testing).

### 6.2 Best First Search (BFS, greedy/OR-graph)
**Idea:** Combine DFS (follow a single path) and BFS (can switch paths). At every step, expand the **most promising** node chosen by the evaluation function **f(n) = h(n)**.

**Data structures:**
- **OPEN**: nodes generated but not yet expanded (priority queue sorted by *h*).
- **CLOSED**: nodes already expanded.

**Algorithm:**
1. Put start node in OPEN.
2. Until goal found or OPEN empty: pick the node with the best (lowest) *h* from OPEN.
3. If it is the goal → success. Otherwise, expand it, move it to CLOSED, add children to OPEN (if not already present).

**Pros:** faster than blind search; can recover from poor choices (unlike hill climbing).
**Cons:** not optimal; not always complete (can follow misleading heuristics); needs memory for OPEN/CLOSED.

### 6.3 Branch and Bound
**Idea:** Explore paths in order of **cost so far** *g(n)* (lowest first). Keep the cost of the best complete path found (the *bound*) and **prune** any partial path whose cost already exceeds it.

**Algorithm:**
1. Initialise the bound = ∞ (or cost of any known solution).
2. Expand the lowest-cost partial path.
3. If a path reaches the goal with cost < bound → update the bound.
4. Discard any partial path with cost ≥ bound.
5. Stop when all remaining paths are pruned/expanded; the bound is the optimal cost.

- With an **underestimate** of remaining cost *h'(n)*: use *f(n) = g(n) + h'(n)* to prune earlier (Branch and Bound with underestimates, the basis of A\*).
- With **dynamic programming** (discard redundant paths to the same node), efficiency improves further.

**Pros:** guarantees optimal solution. **Cons:** can be slow when many partial paths have similar cost.

### 6.4 A\* Algorithm
**Idea:** Best-first search using both the cost already spent and the estimated cost to go.

**f(n) = g(n) + h(n)**
- **g(n)**: actual cost of the path from start to *n*.
- **h(n)**: estimated cost from *n* to goal.
- **f(n)**: estimated total cost of the cheapest solution through *n*.

**Algorithm:**
1. Put start node in OPEN with *f = g + h*.
2. If OPEN is empty → fail.
3. Remove the node *n* with the **lowest f** from OPEN; place it in CLOSED. If *n* is the goal → success (trace path back through parents).
4. Generate successors of *n*. For each successor *s*:
   - Compute g(s) = g(n) + cost(n, s) and f(s) = g(s) + h(s).
   - If *s* is new → add to OPEN, set parent = *n*.
   - If *s* is already in OPEN/CLOSED with a **higher g** → update to the cheaper path (re-open if in CLOSED).
5. Go to step 2.

**Admissibility:** A heuristic is *admissible* if h(n) ≤ true cost from *n* to goal (never overestimates). If *h* is admissible, **A\* is complete and optimal**.
**Consistency (monotonicity):** h(n) ≤ cost(n, n′) + h(n′): ensures nodes are never re-opened.

**Special cases:** h = 0 → uniform-cost search (Dijkstra); g = 0 → greedy best-first.

**Pros:** complete, optimal (admissible *h*), optimally efficient. **Cons:** memory-heavy (stores all generated nodes); time exponential in worst case.

### 6.5 AO\* Algorithm (AND-OR graph search)
**AND-OR graph:** A problem-reduction structure where:
- **OR arc/node**: solve **any one** of the alternative subproblems.
- **AND arc/node**: solve **all** the subproblems together.

Used when a problem can be **decomposed** into subproblems (e.g., theorem proving, game trees, planning). Ordinary A\* cannot handle AND arcs since a solution is a *subgraph*, not a single path.

**Cost revision rule:** For a node with AND arc to successors, cost = Σ (cost of successors + arc cost) (e.g., arc cost 1 each); for OR, take the minimum over alternatives. i.e. **f(n) = Σ [ g + h ] over the AND-set**, minimised over OR alternatives.

**Algorithm (two repeated phases):**
1. Initialise graph G with the start node; compute *h*.
2. **Top-down:** follow the currently marked *best* arcs from the start to find the best partial solution graph; pick an unexpanded node *n* on it.
3. Expand *n*: generate successors, compute *h* for each, add them to G.
4. **Bottom-up cost revision:** from *n* propagate revised costs back toward the start; at each ancestor, choose the cheapest arc (AND-arcs sum the costs) and mark it; mark nodes **SOLVED** when all their AND-successors are solved (or a terminal node).
5. Repeat until the start node is labelled SOLVED (or its cost exceeds the limit → unsolvable).

**AO\* optimality:** gives an optimal solution graph if *h* is admissible (underestimates) and the graph is acyclic.

### A\* vs AO\*
| | A\* | AO\* |
|---|---|---|
| Graph type | OR graph | AND-OR graph |
| Solution | A single path | A solution subgraph |
| Problem type | Path finding | Problem decomposition |
| Cost function | f = g + h | Revised bottom-up over AND/OR arcs |
| Backtracking of costs | Parent pointer updates | Cost propagation to ancestors |

---

## 7. Quick Comparison of Search Techniques

| Technique | Type | Data structure | Uses h(n)? | Complete | Optimal |
|---|---|---|---|---|---|
| BFS | Blind | Queue | No | Yes | Yes (equal costs) |
| DFS | Blind | Stack | No | No | No |
| Hill Climbing | Heuristic | None (local) | Yes | No | No |
| Best First | Heuristic | Priority queue | Yes (h) | No | No |
| Branch & Bound | Cost-based | Priority queue | Optional | Yes | Yes |
| A\* | Heuristic | Priority queue | Yes (g+h) | Yes | Yes (admissible h) |
| AO\* | Heuristic (AND-OR) | Graph | Yes | Yes (acyclic) | Yes (admissible h) |

---

## 8. Important Exam Points
1. Define AI; explain Turing test and four approaches.
2. Production system: components + characteristics (monotonic, partially commutative, commutative) with examples.
3. State-space representation of the water-jug problem.
4. BFS vs DFS (algorithm, complexity, comparison).
5. Hill climbing: variants, local maximum / plateau / ridge, remedies.
6. A\* algorithm: f = g + h, admissibility, algorithm steps.
7. AO\* algorithm and AND-OR graph; A\* vs AO\*.
8. Best-first vs Branch and Bound.

---
*Sources: Rich & Knight "Artificial Intelligence" (Ch. 1–3); Russell & Norvig "AIMA"; Nilsson (AO\*, production systems); course slides/online references cross-checked on production-system classes and A\*/AO\* properties.*
