# 📘 DBMS – Unit III Study Notes (MCA)
> **Topics:** Transaction Management | Concurrency Control | File Organization | Indexing & Hashing | B+ Tree

---

## 1. Transaction Management

### 1.1 What is a Transaction?
A **transaction** is a logical unit of work that contains one or more SQL operations, treated as a single indivisible unit.

**Example:**
```
Bank Transfer: A → B (₹500)
  Step 1: Debit A's account by 500
  Step 2: Credit B's account by 500
```
Both steps must succeed or both must fail — this is atomicity.

---

### 1.2 ACID Properties
| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | All or nothing | Transfer completes fully or not at all |
| **Consistency** | DB stays valid before & after | Total balance unchanged after transfer |
| **Isolation** | Transactions don't interfere | T1 & T2 run as if alone |
| **Durability** | Committed changes survive crashes | After commit, data persists even after power failure |

---

### 1.3 Transaction States
```
                  ┌──────────┐
          ┌──────►│  Active  │──────────────────┐
          │       └────┬─────┘                  │ Fail
          │            │ Partial commit          ▼
       Restart    ┌────▼──────────┐        ┌─────────┐
          │       │  Partially    │        │  Failed │
          └───────│  Committed    │        └────┬────┘
                  └────┬──────────┘             │
                       │ Commit                 │ Rollback
                  ┌────▼──────┐           ┌─────▼──────┐
                  │ Committed │           │  Aborted   │
                  └───────────┘           └────────────┘
```

**States explained:**
- **Active** – transaction is executing
- **Partially Committed** – last statement executed, awaiting commit
- **Committed** – successfully completed, changes permanent
- **Failed** – error encountered, cannot proceed
- **Aborted** – rolled back, DB restored to prior state

---

### 1.4 Serializability

When multiple transactions run concurrently, their interleaved execution should produce the same result as some **serial** (one-after-another) execution.

**Types:**
- **Conflict Serializability** – check using a **Precedence Graph (Serialization Graph)**
  - Two operations conflict if: same data item, different transactions, at least one is a WRITE
  - If precedence graph has **no cycle** → conflict serializable ✅
- **View Serializability** – more general than conflict serializability (rarely tested in detail)

**Example:**
```
Schedule S: T1: R(A), T2: R(A), T1: W(A), T2: W(A)
Conflicting pairs: T1:W(A) vs T2:W(A), T2:R(A) vs T1:W(A)
Precedence graph: T1 → T2  (no cycle) → Conflict Serializable ✅
```

---

### 1.5 Recoverability

| Schedule Type | Definition | Recoverable? |
|---|---|---|
| **Recoverable** | T commits only after all Ts it read from commit | ✅ Always safe |
| **Cascadeless** | Reads only committed data | ✅ Avoids cascading rollbacks |
| **Strict** | Can't read or write X until T that wrote X commits | ✅ Easiest recovery |

> **Cascading rollback problem:** If T1 writes, T2 reads from T1, and T1 aborts → T2 must also abort (cascade). Cascadeless schedules prevent this.

---

## 2. Concurrency Control

### 2.1 Why Concurrency Control?
Without it, we face:
- **Lost Update** – T2 overwrites T1's update
- **Dirty Read** – T2 reads uncommitted data of T1 (which later aborts)
- **Unrepeatable Read** – T1 reads X, T2 modifies X, T1 reads X again → different values
- **Phantom Read** – new rows appear between two reads in same transaction

---

### 2.2 Lock-Based Protocols

**Types of Locks:**
| Lock | Symbol | Allows |
|---|---|---|
| **Shared (S)** | S-lock | Multiple reads; no writes |
| **Exclusive (X)** | X-lock | Single write; no other access |

**Lock Compatibility Table:**
```
        S       X
S       ✅      ❌
X       ❌      ❌
```

---

### 2.3 Two-Phase Locking (2PL)

Every transaction has two phases:
1. **Growing Phase** – acquire locks; cannot release any lock
2. **Shrinking Phase** – release locks; cannot acquire new locks

**2PL guarantees conflict serializability ✅**

**Variants:**
| Variant | Rule |
|---|---|
| **Basic 2PL** | Standard growing + shrinking |
| **Strict 2PL** | Hold all X-locks until commit/abort |
| **Rigorous 2PL** | Hold ALL locks (S + X) until commit/abort |
| **Conservative 2PL** | Acquire ALL locks before transaction starts |

> **Note:** 2PL can cause **deadlocks** but guarantees serializability.

---

### 2.4 Deadlock Handling

**Deadlock:** T1 waits for T2, T2 waits for T1 → circular wait.

#### Prevention:
- **Wait-Die (Non-preemptive):** Older transaction waits; younger is rolled back ("dies")
- **Wound-Wait (Preemptive):** Older transaction "wounds" (preempts) younger; younger waits

| Scheme | Older Tx Wants Lock Held by Younger | Younger Tx Wants Lock Held by Older |
|---|---|---|
| **Wait-Die** | Old waits | Young dies (abort) |
| **Wound-Wait** | Old wounds young (young aborts) | Young waits |

#### Detection:
- **Wait-For Graph:** Nodes = transactions; edge T1 → T2 means T1 waits for T2
- If cycle exists → **deadlock detected**
- **Resolution:** Roll back one transaction in the cycle (choose victim by cost)

---

### 2.5 Timestamp Ordering Protocol

Each transaction gets a unique **timestamp TS(T)** when it starts.

**Rules:**
- For `Read(X)` by Ti: if `TS(Ti) < W-timestamp(X)` → abort Ti (too late to read)
- For `Write(X)` by Ti: if `TS(Ti) < R-timestamp(X)` → abort Ti (too late to write)

✅ **Deadlock-free** (no waiting)
❌ **May cause cascading rollbacks** (not recoverable by default)

**Thomas Write Rule:** If `TS(Ti) < W-timestamp(X)`, simply ignore the write (don't abort). Makes it more lenient.

---

### 2.6 Multiple Granularity Locking

Lock granularity levels (from coarse to fine):
```
Database → Table → Page → Row (Tuple)
```

**Intention Locks** allow a transaction to signal intent at a higher level before locking at a lower level:
| Lock | Meaning |
|---|---|
| **IS** | Intent to get S-lock on child |
| **IX** | Intent to get X-lock on child |
| **SIX** | S-lock on this + IX on children |

> Locking at a coarser level = less overhead but less concurrency; finer level = more concurrency but more overhead.

---

### 2.7 Multi-Version Concurrency Control (MVCC)

- Each write creates a **new version** of a data item
- Readers access **older versions** → no read-write conflict
- **Timestamp-based MVCC:** Each version has `W-TS` and `R-TS`
- Used in PostgreSQL, Oracle
- ✅ High read concurrency; ❌ Extra storage for versions

---

## 3. File Organization

How records are stored physically on disk.

### 3.1 Organization of Records in Files

| Type | Description | Best For |
|---|---|---|
| **Heap File** | Records stored in any free space; no order | Bulk inserts, full scans |
| **Sequential File** | Records sorted by a key | Range queries, ordered output |
| **Hashed File** | Bucket assigned by hash function on key | Exact-match lookups |
| **Clustered File** | Related records from different tables stored together | Joins |

**Record Types:**
- **Fixed-length** – each record same size (easy to locate by offset)
- **Variable-length** – records differ in size (use slotted page or pointer arrays)

---

## 4. Indexing

An **index** is a data structure to speed up data retrieval, similar to a book's index.

### 4.1 Types of Indices

| Type | Description |
|---|---|
| **Primary Index** | On ordered file; built on ordering key field |
| **Clustering Index** | On ordered file; built on non-key ordering field |
| **Secondary Index** | On unordered file; can be on any field |
| **Dense Index** | One index entry per record |
| **Sparse Index** | One index entry per block (only for ordered files) |

> **Dense vs Sparse:** Dense uses more space but faster lookup; Sparse saves space, file must be sorted.

### 4.2 Multilevel Index
- Index on the index (index of index)
- Reduces I/O for large files
- Leads to the idea of B+ Trees

---

## 5. Hashing

### 5.1 Static Hashing
- Fixed number of **buckets**
- Hash function: `h(key) = key mod M` → bucket address
- **Overflow** handled by chaining (linked overflow buckets)
- ❌ Poor performance if many overflows; no dynamic resizing

### 5.2 Dynamic Hashing (Extendible Hashing)
- Hash table grows/shrinks dynamically
- Uses a **directory** of pointers to buckets
- **Global depth** – bits used to index directory
- **Local depth** – bits used by individual bucket
- When bucket overflows → **split** bucket, double directory if needed

**Example:**
```
Global depth = 2 → directory has 4 entries (00, 01, 10, 11)
If bucket for '01' overflows → split; increase local depth to 3
```

---

## 6. B+ Tree Indexing

### 6.1 Structure
- **Balanced tree** where all data pointers are in **leaf nodes**
- Internal nodes hold only keys for routing
- Leaf nodes are **linked** (for range queries)

```
           [30 | 60]           ← Internal node
          /    |    \
   [10|20]  [30|40|50]  [60|70|80]   ← Leaf nodes (linked →)
```

### 6.2 Properties
| Property | Value |
|---|---|
| **Order (n)** | Max n pointers per node |
| **Leaf node** | ≥ ⌈(n-1)/2⌉ keys |
| **Internal node** | ≥ ⌈n/2⌉ pointers |
| **Root** | At least 2 children (if not leaf) |

### 6.3 Operations

**Search:** Follow root → internal nodes → leaf node (O(log n))

**Insertion:**
1. Find correct leaf
2. Insert key in leaf
3. If leaf overflows (n keys) → **split**: promote middle key to parent
4. If parent overflows → split parent too (propagate up)

**Deletion:**
1. Find and remove key from leaf
2. If leaf underflows → **borrow** from sibling OR **merge** with sibling
3. Update parent keys accordingly

**Example – Insert 25 into B+ Tree of order 3:**
```
Before: [10|20] → [30|40]
Insert 25 → [10|20|25] → overflows → split → promote 20
After:     [20]
          /    \
       [10]   [25|30|40]
```

### 6.4 B-Tree vs B+ Tree
| Feature | B-Tree | B+ Tree |
|---|---|---|
| Data pointers | Internal + Leaf nodes | Leaf nodes only |
| Leaf linking | No | Yes (linked list) |
| Range queries | Slow | Fast ✅ |
| Space usage | Less | Slightly more |

> **B+ Tree is preferred for databases** due to efficient range queries and sequential access.

---

## 7. Quick Revision Table

| Concept | Key Point |
|---|---|
| ACID | Atomicity, Consistency, Isolation, Durability |
| 2PL | Growing + Shrinking phases → ensures serializability |
| Strict 2PL | Holds X-locks till commit → prevents dirty reads |
| Deadlock prevention | Wait-Die or Wound-Wait using timestamps |
| Deadlock detection | Wait-For Graph; cycle = deadlock |
| Timestamp ordering | Older TS = higher priority; no deadlocks |
| B+ Tree | Balanced; data only at leaves; leaf nodes linked |
| Dense index | Entry per record; faster but larger |
| Sparse index | Entry per block; smaller but file must be sorted |
| MVCC | Multiple versions; readers don't block writers |

---

## 8. Likely Exam Questions

1. Explain ACID properties with examples.
2. What are the different states of a transaction? Draw the state diagram.
3. Explain Two-Phase Locking. What are its variants?
4. Differentiate between Wait-Die and Wound-Wait deadlock prevention schemes.
5. Explain Timestamp Ordering Protocol with an example.
6. What is Multiple Granularity Locking? Explain intention locks.
7. Explain different file organization techniques.
8. Differentiate between dense and sparse indexing with examples.
9. Explain B+ Tree structure. Illustrate insertion and deletion with an example.
10. Compare B-Tree and B+ Tree.
11. What is extendible hashing? How does it differ from static hashing?
12. Define serializability. How is conflict serializability checked using a precedence graph?

---

*📌 Tip: For numerical questions on B+ trees, always show the tree before and after the operation step-by-step. For serializability, always draw the precedence graph.*
