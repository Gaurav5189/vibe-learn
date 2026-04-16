# 📘 DBMS – Unit IV Study Notes (MCA)
> **Topics:** Query Processing & Optimization | Database Recovery | Database System Architecture

---

## 1. Query Processing

**Query Processing** is the set of steps the DBMS uses to translate a high-level SQL query into an efficient execution plan and retrieve results.

### 1.1 Steps in Query Processing

```
SQL Query
    │
    ▼
┌─────────────┐
│  Parsing &  │  → Checks syntax, validates table/column names
│  Translation│  → Converts SQL to Relational Algebra expression
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Optimization│  → Chooses the most efficient execution plan
│             │  → Uses cost estimates & heuristics
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Execution  │  → Runs the chosen plan, returns result
└─────────────┘
```

---

### 1.2 Basic Algorithms for Query Operations

#### Selection (σ)

| Method | Condition | Cost |
|---|---|---|
| **Linear Search** | No index; any condition | O(b) blocks |
| **Binary Search** | File sorted on selection attribute | O(log₂ b) |
| **Primary Index + equality** | Key attribute with primary index | O(h_i + 1) |
| **Secondary Index + equality** | Non-key attribute | O(h_i + n) |
| **Index + range query** | `A > v` or `A < v` | O(h_i + b) |

> b = number of blocks; h_i = height of index tree

---

#### Sorting (used in ORDER BY, JOIN, GROUP BY)

- **External Sort-Merge** used when data doesn't fit in memory
- **Phase 1 (Run Generation):** Load M blocks at a time, sort in memory, write sorted "runs"
- **Phase 2 (Merging):** Merge sorted runs using (M−1)-way merge

**Cost:**
```
Total I/O = 2b × (⌈log_{M-1}(b/M)⌉ + 1)
```

---

#### Join Algorithms

| Algorithm | Description | Best When |
|---|---|---|
| **Nested Loop Join** | For each tuple in R, scan all of S | Small relations |
| **Block Nested Loop Join** | For each block of R, scan all blocks of S | Buffer limited |
| **Index Nested Loop Join** | Use index on S for each tuple of R | Index on join attr of S |
| **Sort-Merge Join** | Sort both R and S, then merge | Both already sorted / large relations |
| **Hash Join** | Hash both R and S on join attr; match buckets | Large unsorted relations |

**Block Nested Loop cost:**
```
Cost = b_R + (b_R / (M-2)) × b_S
```
where M = available memory blocks

---

#### Projection (π)
- Remove duplicate tuples after projecting columns
- **Method:** Sort on projected attributes → scan and remove duplicates
- Or use **hashing** to detect duplicates

---

#### Set Operations (∪, ∩, −)
- Sort both relations on all attributes
- Scan both simultaneously (like sort-merge) to compute union/intersection/difference
- Alternatively, use **hashing** for partitioning

---

## 2. Query Optimization

The optimizer selects the **cheapest** execution plan from many equivalent plans.

### 2.1 Heuristic-Based Optimization (Rule-Based)

Applies transformation rules to reduce intermediate result sizes **before** cost estimation.

**Key Heuristics / Rules:**

| Rule | Meaning |
|---|---|
| **Cascade of σ** | σ_{c1 AND c2}(R) ≡ σ_{c1}(σ_{c2}(R)) — split conjunctive selections |
| **Commutativity of σ** | σ_{c1}(σ_{c2}(R)) ≡ σ_{c2}(σ_{c1}(R)) |
| **Push Selection Down** ⭐ | Apply σ as early as possible to reduce tuple count |
| **Push Projection Down** ⭐ | Apply π early to eliminate unwanted columns |
| **Commutativity of ⋈** | R ⋈ S ≡ S ⋈ R |
| **Associativity of ⋈** | (R ⋈ S) ⋈ T ≡ R ⋈ (S ⋈ T) — reorder joins |
| **Convert σ + × to ⋈** | σ_c(R × S) ≡ R ⋈_c S (replace Cartesian product with join) |

**Typical Heuristic Strategy:**
1. Break conjunctive selections into cascaded selections
2. Push selections as far down the query tree as possible
3. Push projections down, keeping only needed attributes
4. Identify subtrees that can be pipelined together
5. Order joins to minimize intermediate result sizes

**Example:**
```sql
SELECT name FROM Employee, Department
WHERE Employee.dno = Department.dnumber AND dname = 'IT'
```
**Without optimization:** Employee × Department → filter dno=dnumber → filter dname='IT' → project name
**With push-down:** Filter dname='IT' first (small result) → join → project name  ✅ Much faster
```
π_name (σ_{dno=dnumber} (Employee ⋈ σ_{dname='IT'}(Department)))
```

---

### 2.2 Selectivity & Cost Estimates

The optimizer estimates the **cost** of each plan using statistics stored in the **system catalog**.

#### Statistics Used:
| Statistic | Symbol | Meaning |
|---|---|---|
| Number of tuples | n(R) | Cardinality of relation R |
| Number of blocks | b(R) | Disk blocks occupied |
| Blocking factor | bfr | Tuples per block |
| Number of distinct values | V(A, R) | Distinct values of attribute A in R |
| Selection cardinality | s | Expected tuples satisfying a condition |

#### Selectivity (sel):
```
sel = (number of tuples satisfying condition) / n(R)
```
- Selectivity close to 0 = highly selective (few tuples pass) ✅
- Selectivity close to 1 = low selectivity (most tuples pass)

#### Common Estimates:
| Condition | Estimated Result Size |
|---|---|
| `A = value` | n(R) / V(A, R) |
| `A > value` | n(R) × (max(A) − value) / (max(A) − min(A)) |
| `A < value` | n(R) × (value − min(A)) / (max(A) − min(A)) |
| `R ⋈ S` on A | n(R) × n(S) / max(V(A,R), V(A,S)) |

#### Cost Factors:
- **Disk I/O** (dominant cost — number of block reads/writes)
- CPU cost
- Communication cost (in distributed DBs)

---

### 2.3 Semantic Query Optimization

Uses **integrity constraints** and **semantic knowledge** about the database to simplify queries.

**Idea:** If a constraint guarantees a condition is always true or never true, the query can be rewritten.

**Examples:**

```sql
-- Constraint: salary > 0 for all employees
SELECT * FROM Employee WHERE salary < 0;
-- Optimizer knows no such tuples exist → returns empty immediately ✅

-- Constraint: Every employee belongs to a department (FK enforced)
SELECT e.name FROM Employee e, Department d WHERE e.dno = d.dnumber;
-- Optimizer may eliminate the join since FK guarantees match ✅
```

**Benefits:**
- Eliminates unnecessary joins
- Detects unsatisfiable queries early
- Reduces query execution cost significantly

---

## 3. Database Recovery

Recovery ensures the database can be restored to a **consistent state** after a failure.

### 3.1 Types of Failures
| Failure Type | Cause | Recovery Method |
|---|---|---|
| **Transaction Failure** | Logic error, abort | Rollback (UNDO) |
| **System Failure** | OS crash, power loss | Redo/Undo using log |
| **Media Failure** | Disk crash | Backup + Archive log |

---

### 3.2 Log-Based Recovery

A **log (journal)** is a sequence of log records written to stable storage **before** any DB modification.

#### Log Record Types:
```
[start,        Ti]           → Transaction Ti has started
[write,        Ti, X, old_val, new_val]  → Ti is writing X
[read,         Ti, X]        → Ti is reading X
[commit,       Ti]           → Ti has committed
[abort,        Ti]           → Ti has aborted
[checkpoint]                 → Checkpoint record
```

#### Write-Ahead Logging (WAL) Rule ⭐
> **Before** a data item is written to disk, its log record **must** be written to stable storage.

This ensures we can UNDO a transaction if needed.

---

#### Deferred Update (NO-UNDO / REDO)
- DB is **not modified** until a transaction commits
- Log records all changes; writes to DB only at commit
- On failure: simply ignore uncommitted transactions (no UNDO needed)
- On recovery: **REDO** committed transactions whose writes didn't reach disk

#### Immediate Update (UNDO / REDO)
- DB is modified **before** commit (write-ahead log required)
- On failure: **UNDO** uncommitted transactions, **REDO** committed ones
- More complex but allows writing to disk any time

---

#### Recovery Algorithm (UNDO/REDO)

**Phase 1 – Analysis:** Scan log from last checkpoint → determine which Ts were active at crash

**Phase 2 – REDO:** Redo all operations from the oldest log record of an active transaction (restores DB to crash state)

**Phase 3 – UNDO:** Undo operations of transactions that were active at crash (roll them back)

```
UNDO list:  transactions that started but did NOT commit before crash
REDO list:  transactions that committed (may need redo if not flushed to disk)
```

---

### 3.3 Checkpointing

A **checkpoint** is a snapshot point where the DBMS forces all dirty buffers to disk and writes a checkpoint record to the log.

**Benefits:**
- Limits how far back recovery needs to scan the log
- Reduces REDO work after a crash

**Checkpoint Process:**
1. Suspend all transactions temporarily
2. Write all dirty (modified) buffer pages to disk
3. Write `[checkpoint]` record to log
4. Resume transactions

**Fuzzy Checkpoint:** Allows transactions to continue while checkpoint is being written (avoids long pauses).

---

### 3.4 Shadow Paging

An alternative to log-based recovery — **no log needed**.

**Concept:**
- Maintain two page tables: **Current Page Table** and **Shadow Page Table**
- Shadow page table = copy of page table at last commit
- Writes go to new (shadow) pages; original pages unchanged
- On **commit:** update current page table → becomes new shadow
- On **abort/crash:** discard current page table, restore from shadow

```
Shadow Page Table ──────────────────────► Old (stable) pages
                                           (never modified during Tx)
Current Page Table ─────┐
                         ▼
                    New/Modified pages
                    (discarded on abort)
```

**Advantages:** Simple recovery (just restore shadow page table)
**Disadvantages:**
- Fragmentation (pages scatter across disk)
- Expensive commit (must copy entire page table)
- Not suitable for multi-user concurrent access

---

### 3.5 Recovery with Concurrent Transactions

When multiple transactions run concurrently, recovery is more complex.

**Issues:**
- Multiple UNDO/REDO lists at the same time
- Need to track which transactions affect which data items

**Solution – ARIES Algorithm** (Algorithm for Recovery and Isolation Exploiting Semantics):

Three phases:
1. **Analysis Phase:** Scan log from last checkpoint → build transaction table and dirty page table
2. **Redo Phase:** Replay all logged actions from oldest dirty page LSN (restore crash state)
3. **Undo Phase:** Undo all incomplete transactions in reverse log order

**LSN (Log Sequence Number):** Unique number assigned to each log record; used to track what has been applied.

> ARIES is used in real systems like IBM DB2 and Microsoft SQL Server.

---

### 3.6 Advanced Recovery Techniques

#### ARIES Optimizations:
| Technique | Description |
|---|---|
| **Compensation Log Records (CLR)** | Written during UNDO; describes what was undone; makes recovery idempotent |
| **Repeating History** | Redo ALL actions (even of aborted Txs) before undoing — simplifies logic |
| **Selective Redo** | Only redo operations not yet on disk (using dirty page table) |

#### Media Recovery:
- Use **database dump/backup** + **archive logs**
- Restore from last backup, then replay archive logs up to failure point

#### High Availability:
- **Mirroring / RAID** – redundant disks to protect against media failure
- **Remote Backup** – secondary DB on a different site; log records shipped continuously
  - On primary failure, secondary takes over (failover)

---

## 4. Database System Architecture

### 4.1 Centralized Architecture

All DBMS functions run on a **single machine** (mainframe or server).

```
┌────────────────────────────┐
│     Single Central Server  │
│  ┌───────┐  ┌───────────┐  │
│  │  CPU  │  │ DBMS + DB │  │
│  └───────┘  └───────────┘  │
└────────────┬───────────────┘
             │ Terminals / Thin clients
             ▼
     [User 1] [User 2] [User 3]
```

**Advantages:** Simple management, easy security, consistent data
**Disadvantages:** Single point of failure, scalability limited, high server load

---

### 4.2 Client-Server Architecture

Processing is split between **clients** (front-end) and **servers** (back-end).

```
[Client 1] ──┐
[Client 2] ──┼──► [Database Server]
[Client 3] ──┘
```

#### Two-Tier Architecture:
- Client communicates **directly** with DB server
- Business logic on client side
- Simple but tight coupling

#### Three-Tier Architecture (Most Common Today):
```
[Client / Browser]
        │  HTTP
        ▼
[Application Server]   ← Business logic here
        │  SQL
        ▼
[Database Server]      ← Data storage & retrieval
```
**Advantages:** Scalability, security (DB not directly exposed), separation of concerns

---

### 4.3 Server System Architecture

| Type | Description |
|---|---|
| **Transaction Server** | Clients send transactions; server processes and returns result (most DB systems) |
| **Data Server** | Clients request raw data pages; client processes the query |

> Transaction servers are more efficient as less data is transferred over network.

---

### 4.4 Parallel Database Systems

Use **multiple processors and disks** to improve performance.

**Goals:** High throughput (more queries/sec) and low response time.

#### Types of Parallelism:

| Type | Description |
|---|---|
| **Inter-query parallelism** | Different queries run on different processors |
| **Intra-query parallelism** | Single query split across multiple processors |
| **Intra-operation parallelism** | Single operation (e.g., sort) done in parallel |
| **Inter-operation parallelism** | Different operations of same query run in parallel (pipelining) |

#### Parallel System Architectures:

| Architecture | Memory | Disk | Description |
|---|---|---|---|
| **Shared Memory** | Shared | Shared | All CPUs access same memory & disks; fast communication; limited scalability |
| **Shared Disk** | Private | Shared | Each CPU has own memory; all share same disk; good fault tolerance |
| **Shared Nothing** ⭐ | Private | Private | Each node fully independent; communicates via network; best scalability (used in Hadoop, Google Spanner) |
| **Hierarchical** | Mixed | Mixed | Combination: shared memory within nodes, shared nothing between nodes |

```
Shared Nothing Architecture:
[Node 1: CPU+Mem+Disk] ──Network── [Node 2: CPU+Mem+Disk]
                           │
               [Node 3: CPU+Mem+Disk]
```

---

### 4.5 Distributed Database Systems

Data is stored across **multiple geographically distributed sites**, connected by a network.

```
[Site A: Delhi]  ──── Network ────  [Site B: Mumbai]
      │                                     │
   DB_A                                  DB_B
```

#### Key Properties:

| Property | Description |
|---|---|
| **Data Distribution** | Data physically at different sites |
| **Data Replication** | Same data kept at multiple sites for availability |
| **Data Fragmentation** | A table split across sites |
| **Transparency** | User sees one logical DB regardless of distribution |

#### Fragmentation Types:
| Type | Description | Example |
|---|---|---|
| **Horizontal** | Rows split across sites | Employees from North India at Site A, South India at Site B |
| **Vertical** | Columns split across sites | Employee ID, Name at Site A; Salary, Dept at Site B |
| **Mixed** | Combination of both | — |

#### Advantages of Distributed DB:
- High availability (if one site fails, others serve data)
- Better performance (local queries)
- Scalability

#### Challenges:
- **Distributed transactions** – need 2-Phase Commit (2PC) protocol
- **Distributed query optimization** – must minimize data transfer
- **Consistency** – ensuring all replicas are in sync (CAP Theorem)

#### Two-Phase Commit (2PC): ⭐
Ensures atomicity in distributed transactions.

```
Phase 1 – Prepare:
  Coordinator → "Prepare to commit" → all Participants
  Participants reply "YES (ready)" or "NO (abort)"

Phase 2 – Commit/Abort:
  If all YES → Coordinator sends "COMMIT" to all
  If any NO  → Coordinator sends "ABORT" to all
```

**Problem:** If coordinator crashes after participants vote YES but before sending COMMIT → participants are **blocked** (waiting indefinitely). This is the **blocking problem** of 2PC.

#### CAP Theorem:
A distributed system can guarantee **only 2 of 3**:
- **C**onsistency – all nodes see same data
- **A**vailability – every request gets a response
- **P**artition Tolerance – system works despite network partition

---

## 5. Quick Revision Table

| Concept | Key Point |
|---|---|
| Query Processing Steps | Parse → Optimize → Execute |
| Push Selection Down | Key heuristic — apply σ early to reduce tuples |
| Selectivity | sel = matching tuples / total tuples; lower = better |
| WAL Rule | Log must be written before data reaches disk |
| Deferred Update | No-Undo/Redo; DB modified only at commit |
| Immediate Update | Undo/Redo; DB can be modified before commit |
| Checkpoint | Flushes dirty pages; limits log scan on recovery |
| Shadow Paging | Two page tables; no log needed; restore shadow on failure |
| ARIES | Analysis → Redo → Undo; uses LSN; real-world systems |
| Shared Nothing | Each node independent; best scalability (Hadoop, Spanner) |
| 2PC | Distributed commit protocol; Phase 1=Prepare, Phase 2=Commit |
| CAP Theorem | Only 2 of Consistency, Availability, Partition Tolerance |

---

## 6. Likely Exam Questions

1. Explain the steps in query processing with a diagram.
2. What are the basic algorithms used for the Selection operation in query processing?
3. Explain heuristic optimization with the "push selection down" rule using an example.
4. What is selectivity? How is cost estimated in query optimization?
5. What is semantic query optimization? Give examples.
6. Explain Write-Ahead Logging (WAL). What is deferred vs immediate update?
7. What is checkpointing? How does it help in recovery?
8. Explain shadow paging. What are its advantages and disadvantages?
9. Explain the ARIES recovery algorithm with its three phases.
10. What is Two-Phase Commit (2PC)? What is the blocking problem?
11. Compare Shared Memory, Shared Disk, and Shared Nothing architectures.
12. Explain horizontal and vertical fragmentation in distributed databases.
13. What is the CAP theorem? Explain with examples.
14. Compare centralized, client-server, and distributed database architectures.

---

*📌 Tip: For query optimization questions, always show the query tree before and after applying heuristics. For recovery questions, show example log records and explain REDO/UNDO lists clearly.*
