# Artificial Intelligence (MCC-2.3.4) — Unit II Notes
**PG MCA, Ravenshaw University (CBCS)** | Ref: Rich & Knight (Ch. 12.1–12.3, 15.1–15.4 and related); Patterson; Russell & Norvig

**Unit II syllabus:** Game playing: Min-Max Search, Alpha-Beta Cutoff · Knowledge Logic: Skolemizing queries, Unification algorithm, Modus Ponens, Resolution · Structured knowledge representation: Semantic nets, Frames, Conceptual dependencies, Scripts

---

# PART A — GAME PLAYING

## 1. Introduction
**Game playing** is search in an **adversarial** setting: two players (MAX and MIN) alternate moves, each trying to win. Games are a classic AI domain because they are well-defined, easy to evaluate, but have huge search spaces (chess ≈ 35^100 nodes).

**Game types:** deterministic vs. chance; perfect vs. imperfect information. Here: **two-player, zero-sum, deterministic, perfect-information** games (chess, tic-tac-toe, checkers).

**Game tree components:**
- **Initial state**: starting board + player to move.
- **Successor function**: legal moves.
- **Terminal test**: game over?
- **Utility function**: final payoff (+1 win, 0 draw, −1 loss).
- **MAX** tries to maximise the score, **MIN** tries to minimise it.
- **Ply**: one move by one player (half-move).

**Static evaluation function:** estimates the goodness of a board position when the search is cut off at a depth limit (e.g., chess: material count + position). Positive = good for MAX.

---

## 2. Min-Max Search

**Idea:** Assume the opponent plays optimally. Choose the move that maximises the minimum payoff MAX can be forced into.

**Rules:**
- At **MAX** nodes: take the **maximum** of children's values.
- At **MIN** nodes: take the **minimum** of children's values.
- Values propagate **bottom-up** from the leaves (depth-first traversal).

**Algorithm (recursive):**
```
MINIMAX(node, depth, player):
    if depth == 0 or node is terminal:
        return STATIC_EVAL(node)
    if player == MAX:
        best = -∞
        for each child: best = max(best, MINIMAX(child, depth-1, MIN))
        return best
    else:
        best = +∞
        for each child: best = min(best, MINIMAX(child, depth-1, MAX))
        return best
```

**Example:** MAX root, three MIN children, leaf values:
```
            MAX
        /    |    \
     MIN    MIN    MIN
    /|\     /|\     /|\
   3 12 8  2 4 6  14 5 2
```
MIN values = 3, 2, 2 → MAX picks **3** (left move).

| Property | Value |
|---|---|
| Complete | Yes (finite tree) |
| Optimal | Yes (against an optimal opponent) |
| Time | O(b^m) |
| Space | O(b·m) (depth-first) |

**Limitation:** explores the entire tree to the depth limit; exponential time, which is impractical for large games → improved by Alpha-Beta pruning.

---

## 3. Alpha-Beta Cutoff (Pruning)

**Idea:** Same result as Min-Max but **skips (prunes) branches** that cannot affect the final decision.

- **α (alpha)**: best (highest) value found so far for **MAX** along the path (lower bound; initially −∞).
- **β (beta)**: best (lowest) value found so far for **MIN** along the path (upper bound; initially +∞).

**Cutoff rules:**
- **β-cutoff (at MAX node):** if value ≥ β → stop exploring remaining children.
- **α-cutoff (at MIN node):** if value ≤ α → stop exploring remaining children.
- Prune whenever **α ≥ β**.

**Algorithm:**
```
ALPHABETA(node, depth, α, β, player):
    if depth == 0 or terminal: return STATIC_EVAL(node)
    if player == MAX:
        for each child:
            α = max(α, ALPHABETA(child, depth-1, α, β, MIN))
            if α >= β: break          # β cutoff
        return α
    else:
        for each child:
            β = min(β, ALPHABETA(child, depth-1, α, β, MAX))
            if α >= β: break          # α cutoff
        return β
```

**Example (same tree):**
1. Left MIN node: sees 3, 12, 8 → value 3. Root α = 3.
2. Middle MIN node: first leaf = 2 → 2 ≤ α(3), so **prune the remaining leaves (4, 6)**.
3. Right MIN node: 14, 5, 2 → value 2. 
4. Root = **3**. Same answer, fewer nodes evaluated.

**Effectiveness:**
- Result is **identical** to Min-Max.
- **Worst case** (bad move ordering): no pruning, O(b^m).
- **Best case** (perfect ordering): O(b^(m/2)), i.e. search **twice as deep** in the same time.
- Good move ordering (try best moves first) is key.

| | Min-Max | Alpha-Beta |
|---|---|---|
| Nodes examined | All | Fewer (pruned) |
| Result | Optimal | Same optimal result |
| Extra info | None | α and β bounds |
| Best-case time | O(b^m) | O(b^(m/2)) |

---

# PART B — KNOWLEDGE LOGIC

## 4. Logic as a Knowledge Representation Language

**Why logic?** Precise syntax and semantics; supports sound **inference** (deriving new facts).

### 4.1 Propositional logic (recap)
- **Proposition:** a statement that is true or false (e.g., "It is raining").
- Connectives: ¬ (not), ∧ (and), ∨ (or), → (implies), ↔ (iff).
- Limitation: cannot express general statements about objects ("All men are mortal").

### 4.2 Predicate (First-Order) logic (recap)
Adds: **constants** (Marcus), **variables** (x), **predicates** (Man(x)), **functions** (father(x)), **quantifiers**: ∀ (for all), ∃ (there exists).
- "All men are mortal" → ∀x Man(x) → Mortal(x)
- "Someone loves everyone" → ∃x ∀y Loves(x, y)

**Inference** = deriving new sentences from the knowledge base using sound rules.

---

## 5. Modus Ponens

**Rule:** If **P** is true and **P → Q** is true, then **Q** is true.

```
P,  P → Q
---------
    Q
```

**Example:** 
- "If it rains, the ground is wet" (Rain → Wet); "It rains" (Rain) ⊢ **Wet**.
- Predicate form: Man(Marcus); ∀x Man(x) → Mortal(x) ⊢ Mortal(Marcus) (after substituting x = Marcus).

**Generalised Modus Ponens (first-order):** For atoms p_i, p_i′ and substitution θ such that SUBST(θ, p_i′) = SUBST(θ, p_i) for all i:
```
p1', p2', ..., pn',  (p1 ∧ p2 ∧ ... ∧ pn → q)
------------------------------------------------
                 SUBST(θ, q)
```
- **Sound** and used in **forward/backward chaining** with Horn clauses.
- **Limitation:** works only on Horn-clause-style implications; not complete for full FOL. **Resolution** is more general.

---

## 6. Clause Form and Skolemizing

**Why?** Resolution works only on sentences in **clause form**: a conjunction of clauses, where each **clause** is a disjunction of literals (e.g., ¬Man(x) ∨ Mortal(x)).

### 6.1 Skolemization
**Skolemization** eliminates **existential quantifiers** by replacing each existentially quantified variable with:
- a **Skolem constant** if no universal quantifier encloses it, or
- a **Skolem function** of the enclosing universally quantified variables.

| Original | Skolemized | Reason |
|---|---|---|
| ∃x P(x) | P(c) | c = new constant (the "something that exists") |
| ∀x ∃y Loves(x, y) | ∀x Loves(x, f(x)) | y depends on x → function f(x) |
| ∀x ∀y ∃z R(x, y, z) | ∀x ∀y R(x, y, g(x, y)) | z depends on both |

Skolem names must be **new** (not used elsewhere). Skolemization preserves *satisfiability* (not equivalence).

### 6.2 Skolemizing queries
In resolution **refutation**, the **query (goal) is negated** and added to the knowledge base, then converted to clause form:
- Query: ∃x Likes(x, Pizza)? Negation: ∀x ¬Likes(x, Pizza) → clause ¬Likes(x, Pizza) (no Skolemization needed).
- Query: ∀x Loves(x, Mary)? Negation: ∃x ¬Loves(x, Mary) → **Skolemize**: ¬Loves(c, Mary), where **c** is a Skolem constant.

**Key point:** Negating flips quantifiers (∀ ↔ ∃), so a universal query yields an existential after negation, which then needs Skolemization.

### 6.3 Conversion to clause form (Rich & Knight, 9 steps)
1. **Eliminate →** : P → Q ≡ ¬P ∨ Q.
2. **Move ¬ inward** (De Morgan, ¬∀x ≡ ∃x¬, ¬∃x ≡ ∀x¬, ¬¬P ≡ P).
3. **Standardise variables** (each quantifier uses a distinct variable name).
4. **Move all quantifiers to the left** (prenex form), keeping order.
5. **Eliminate existential quantifiers** (Skolemization).
6. **Drop the universal prefix** (all remaining variables are universal).
7. **Convert matrix to conjunction of disjunctions** (CNF; distribute ∨ over ∧).
8. **Create a separate clause for each conjunct.**
9. **Standardise variables apart** across clauses (each clause gets its own variable names).

**Example:** ∀x [Man(x) → Mortal(x)]
→ ∀x [¬Man(x) ∨ Mortal(x)] → clause: **¬Man(x) ∨ Mortal(x)**

---

## 7. Unification Algorithm

**Unification:** The process of finding a **substitution** that makes two logical expressions **identical**. Needed so resolution can match a literal with the negation of another.

- **Substitution** θ = {v₁/t₁, v₂/t₂, …}: replace variable vᵢ by term tᵢ.
- **Unifier:** a substitution that makes the expressions identical.
- **Most General Unifier (MGU):** the least restrictive unifier (others are specialisations of it).

**Examples:**
| Expression 1 | Expression 2 | Result |
|---|---|---|
| P(x) | P(A) | {A/x} |
| Knows(John, x) | Knows(John, Jane) | {Jane/x} |
| P(x, f(y)) | P(a, f(b)) | {a/x, b/y} |
| P(x, x) | P(a, b) | **Fail** (x cannot be both a and b) |
| P(x) | P(f(x)) | **Fail** (occurs check: x occurs in f(x)) |
| P(a) | Q(a) | **Fail** (different predicates) |

**Algorithm Unify(L1, L2) (Rich & Knight):**
1. If L1 or L2 is a **variable or constant**:
   - (a) If L1 and L2 are identical → return NIL (success, no substitution).
   - (b) Else if L1 is a variable: if L1 occurs in L2 → return FAIL; else return {L2/L1}.
   - (c) Else if L2 is a variable: if L2 occurs in L1 → return FAIL; else return {L1/L2}.
   - (d) Else return FAIL.
2. If the **predicate symbols** (or function symbols) of L1 and L2 differ → FAIL.
3. If L1 and L2 have a **different number of arguments** → FAIL.
4. Set SUBST = NIL.
5. For i = 1 to number of arguments:
   - (a) Call Unify on the ith arguments of L1 and L2; put result in S.
   - (b) If S = FAIL → return FAIL.
   - (c) If S ≠ NIL: apply S to the remainder of both L1 and L2; SUBST = APPEND(S, SUBST).
6. Return SUBST.

**Occurs check:** prevents infinite structures like x = f(x).

---

## 8. Resolution

**Resolution** is a single, sound, refutation-complete inference rule for clause-form sentences. 

### 8.1 Propositional resolution
From clauses (A ∨ B) and (¬B ∨ C), derive the **resolvent** (A ∨ C).
```
 A ∨ B ,  ¬B ∨ C
 ----------------
      A ∨ C
```
Special cases: P and ¬P resolve to the **empty clause (□)**, which is a contradiction.
(Modus Ponens is a special case: P and ¬P ∨ Q give Q.)

### 8.2 Predicate (First-order) resolution
Two clauses resolve if one contains literal **L** and the other contains **¬M**, where L and M **unify** with MGU θ. The resolvent = union of the remaining literals, with θ applied.

```
 P(x) ∨ Q(x),   ¬P(a) ∨ R(y)        θ = {a/x}
 --------------------------------
          Q(a) ∨ R(y)
```

### 8.3 Resolution refutation (proof by contradiction) — algorithm
To prove statement **P** from a set of axioms **F**:
1. Convert all axioms in F to **clause form**.
2. **Negate P** and convert it to clause form (skolemize); add to the clause set.
3. Repeat until the **empty clause** is derived or no progress:
   - (a) Select two clauses (parent clauses).
   - (b) Resolve them (using unification) to get the resolvent.
   - (c) If the resolvent is the empty clause → **P is proved**.
   - (d) Else add the resolvent to the clause set.
4. If no new clauses can be produced → **P cannot be proved** (may loop forever for FOL since it is semi-decidable).

### 8.4 Worked example
**Facts:** (1) Marcus is a man. (2) All men are mortal. **Prove:** Marcus is mortal.

*Clause form:*
1. Man(Marcus)
2. ¬Man(x) ∨ Mortal(x)
3. ¬Mortal(Marcus)  ← negated goal

*Resolution:*
- Resolve (2) and (3) with {Marcus/x} → **¬Man(Marcus)** 
- Resolve ¬Man(Marcus) with (1) → **□ (empty clause)**
- Contradiction → the original goal **Mortal(Marcus)** is proved.

### 8.5 Resolution strategies (to reduce search)
- **Unit preference:** prefer resolving with single-literal clauses.
- **Set of support:** at least one parent must come from the negated goal (or its descendants).
- **Input resolution:** one parent is always an original clause.
- **Linear resolution:** new clause is always resolved against the previous resolvent.

### 8.6 Properties
- **Sound**: derives only valid conclusions.
- **Refutation-complete**: if the set is unsatisfiable, resolution derives □.
- **Semi-decidable** for FOL: terminates if a proof exists, but may run forever otherwise.

### Modus Ponens vs Resolution
| | Modus Ponens | Resolution |
|---|---|---|
| Input form | Implications (Horn clauses) | Any clause form |
| Direction | Forward/backward chaining | Refutation |
| Completeness | Complete for Horn clauses only | Refutation-complete for all FOL |
| Needs unification | Generalised MP: yes | Yes |

---

# PART C — STRUCTURED KNOWLEDGE REPRESENTATION

**Why structured?** Logic stores knowledge as unrelated facts. Structured representations **group related knowledge together**, enable **inheritance**, and mirror how humans organise knowledge.

## 9. Semantic Nets (Semantic Networks)

**Definition:** A **graph** for representing knowledge: **nodes** = objects/concepts; **labelled arcs (links)** = relationships between them. (Quillian, 1968.)

**Common links:**
- **isa** (subclass): Dog isa Mammal
- **instance** (member): Tommy instance Dog
- **has-part**, **color**, **owner**, etc.

**Example:**
```
Animal ──has──> Skin
  ▲ isa
Mammal ──has──> Fur ... warm-blooded
  ▲ isa
Dog ──sound──> Barks
  ▲ instance
Tommy ──color──> Brown
```
- "Tommy is a dog" (instance), "A dog is a mammal" (isa), so Tommy inherits mammal properties.

**Inheritance:** Lower nodes inherit properties of higher nodes via isa/instance links. Specific values **override** inherited defaults (e.g., Penguin: can-fly = No overrides Bird: can-fly = Yes).

**Inference by traversal:** To answer "Does Tommy have fur?", follow Tommy → Dog → Mammal → has Fur.

| Advantages | Disadvantages |
|---|---|
| Visual, intuitive | No standard formal semantics |
| Efficient inheritance & retrieval | Hard to represent negation, quantifiers, disjunction |
| Natural for hierarchies | Large networks become complex; no standard link names |
| Related facts stored together | Handling exceptions needs care |

**Partitioned semantic nets** (Hendrix) group nodes into spaces to represent quantification and beliefs.

---

## 10. Frames

**Definition:** A **frame** (Minsky, 1975) is a data structure representing a **stereotyped situation or object** as a collection of **slots** (attributes) with **fillers** (values). Similar to **objects/classes in OOP**.

**Structure:**
```
Frame: Dog
  isa:        Mammal
  Slot        Value/Facet
  sound:      Barks             (default)
  legs:       4                 (default)
  color:      [any]             (if-needed: ask owner)
  age:        [0–30]            (range)

Frame: Tommy
  instance-of: Dog
  color:      Brown
  age:        3
```

**Slot facets (properties of slots):**
- **Value:** the actual filler.
- **Default:** assumed if no specific value given.
- **Range/Type constraint:** allowed values.
- **If-needed (procedural attachment):** procedure run to compute a value when needed.
- **If-added:** procedure triggered when a value is filled in.
- **If-removed:** triggered on deletion.

**Key features:**
- **Inheritance:** instance frames inherit slots from class frames (via isa/instance-of).
- **Defaults:** support reasoning with incomplete information; can be overridden.
- **Procedural attachment:** combine declarative data with procedures (demons).
- **Expectation-driven:** a frame activated by observation sets up expectations for missing slots.

| Advantages | Disadvantages |
|---|---|
| Organised, structured knowledge | Hard to design hierarchies for large domains |
| Defaults & inheritance | Not a formal logic; ambiguous semantics |
| Procedural attachments | Handling exceptions/multiple inheritance is complex |
| Natural mapping to OOP | Inference is limited |

**Semantic nets vs Frames:** A frame is essentially a semantic-net **node with its internal structure (slots) made explicit**; frames add defaults and procedures.

---

## 11. Conceptual Dependency (CD)

**Definition:** A knowledge representation (Schank, 1969–73) that represents the **meaning of sentences** in terms of a small set of **primitive actions**, independent of the language/words used.

**Goals:**
- Help in drawing **inferences** from sentences.
- Be **independent of the words** of the input: sentences with the same meaning get **the same representation** (canonical form).
- Used in programs such as **MARGIE, SAM, PAM**.

### 11.1 Conceptual categories (building blocks)
| Symbol | Category | Meaning |
|---|---|---|
| **PP** | Picture Producer | Physical object / actor |
| **ACT** | Action | One of the primitive acts |
| **PA** | Picture Aider | Attribute of a PP (e.g., colour, size) |
| **AA** | Action Aider | Attribute of an ACT (e.g., speed) |
| **T** | Times | Time of action |
| **LOC** | Locations | Place of action |

### 11.2 Primitive ACTs (Schank & Abelson)
| Primitive | Meaning | Example verbs |
|---|---|---|
| **ATRANS** | Transfer of an **abstract relationship** (possession, ownership, control) | give, take, buy |
| **PTRANS** | Transfer of **physical location** of an object | go, move, fly |
| **MTRANS** | Transfer of **mental information** between agents / within a mind | tell, read, see, hear |
| **PROPEL** | Application of **physical force** to an object | push, throw, hit |
| **MOVE** | Movement of a **body part** by its owner | kick, wave |
| **GRASP** | Grasping an object | hold, clutch |
| **INGEST** | Taking something **into the body** | eat, drink |
| **EXPEL** | Expelling something **from the body** | cry, spit |
| **MBUILD** | Building new information from old (thinking) | decide, conclude |
| **ATTEND** | Focusing a sense organ | look, listen |
| **SPEAK** | Producing sounds | say, utter |

### 11.3 Notation and dependencies
- **⇔** : two-way dependency between actor and action.
- **←** : direction of object/recipient.
- **o** over the arrow: **object** case; **R**: recipient (to/from); **I**: instrument; **D**: direction.
- **p** : past tense; **f**: future; **t**: transition, **k**: continuing, **?**: interrogative, **/**: negative.

**Examples:**

1. **"John gave Mary a book."**
```
John ⇔ ATRANS ←o─ book ←R─ ( to: Mary
                                from: John )
```
Meaning: John transferred possession of the book from John to Mary.

2. **"John went to the market."** → John ⇔ PTRANS ←o─ John ←D─ (to: market, from: unknown)

3. **"John told Mary he was hungry."** → John ⇔ MTRANS ←o─ (information) ←R─ (to: Mary's mind, from: John's mind)

4. **"John ate the egg."** → John ⇔ INGEST ←o─ egg ←D─ (to: John's mouth, from: John's hand)

5. **"Buy" = two ATRANS:** money from buyer → seller, and goods from seller → buyer.

| Advantages | Disadvantages |
|---|---|
| Language independent; canonical representation | Primitive set is limited and arbitrary |
| Reduces inference rules (inferences attach to primitives) | Verbose; large number of primitives needed for complex events |
| Helps natural language understanding & paraphrasing | Hard to represent abstract concepts and attitudes |

---

## 12. Scripts

**Definition:** A **script** (Schank & Abelson, 1977) is a **structured representation of a stereotyped sequence of events** in a particular context (e.g., eating at a restaurant, visiting a doctor, going shopping). Scripts organise **CD structures** into typical episodes and let a system **fill in unstated details** when understanding stories.

### 12.1 Components of a script
| Component | Meaning |
|---|---|
| **Entry conditions** | Conditions that must hold before the script can start |
| **Roles** | People involved (Customer, Waiter, Cook, Cashier) |
| **Props** | Objects used (Table, Menu, Food, Bill, Money) |
| **Scenes** | Ordered sequence of events (broken into sub-sequences) |
| **Track** | Specific variation of the general script (e.g., café vs fast food) |
| **Results** | Conditions true after the script ends |

### 12.2 Example: Restaurant script
- **Track:** Coffee shop. **Props:** Tables, Menu, Food, Bill, Money. **Roles:** S = Customer, W = Waiter, C = Cook, M = Cashier, O = Owner.
- **Entry conditions:** S is hungry; S has money.
- **Scene 1 — Entering:** S PTRANS S into restaurant → S ATTEND eyes to tables → S MBUILD where to sit → S PTRANS S to table → S MOVE S to sitting position.
- **Scene 2 — Ordering:** W PTRANS menu to S → S MBUILD choice of food → S MTRANS signal to W → W PTRANS W to table → S MTRANS "I want food" to W → W PTRANS W to C → W MTRANS (ATRANS food) to C.
- **Scene 3 — Eating:** C ATRANS food to W → W ATRANS food to S → S INGEST food.
- **Scene 4 — Exiting:** W ATRANS bill to S → S ATRANS money to W (or M) → S PTRANS S out of restaurant.
- **Results:** S has less money; O has more money; S is not hungry; S is pleased (optional).

**Use in understanding:** Story: "John went to a restaurant. He ordered a hamburger. He paid and left." A script-based system **infers** missing events (sat down, ate, was served).

**Mechanism:** Script is **activated** by key words/phrases (script header); then **instantiated** with story entities; missing events are **inferred**.

| Advantages | Disadvantages |
|---|---|
| Captures typical event sequences and common-sense knowledge | Rigid; cannot handle unexpected/novel events |
| Helps fill in missing information, answer questions | A new script is needed for each situation |
| Good for natural language story understanding | Not suitable for creative or unusual situations; hard to choose the right script |

---

## 13. Comparison of Structured Representations

| Feature | Semantic Net | Frame | Conceptual Dependency | Script |
|---|---|---|---|---|
| Basic unit | Node + labelled arc | Slots and fillers | Primitive ACT + cases | Sequence of CD events |
| Represents | Concepts & relationships | Objects/situations | Meaning of sentences | Stereotyped event sequences |
| Inheritance | Yes (isa) | Yes (isa, defaults) | No | No |
| Main strength | Hierarchies, easy retrieval | Defaults, procedures | Language independence | Context, fill-in gaps |
| Main weakness | No formal semantics | Complex hierarchy design | Limited primitives | Rigid |
| Typical use | Taxonomies, WordNet | Expert systems, OOP | NLU (MARGIE, SAM) | Story understanding |

---

## 14. Important Exam Points
1. Min-Max algorithm with an example tree; properties and limitations.
2. Alpha-Beta pruning: α, β, cutoff conditions, example, best/worst case.
3. Min-Max vs Alpha-Beta.
4. Skolemization (constants vs functions) and Skolemizing queries.
5. Steps to convert a FOL sentence to clause form.
6. Unification algorithm with examples (including failure cases/occurs check).
7. Modus Ponens (and generalised MP).
8. Resolution: algorithm, refutation proof of a given example.
9. Semantic nets and frames: structure, inheritance, advantages/disadvantages.
10. Conceptual dependency: primitives (ATRANS, PTRANS, MTRANS...), represent a sentence.
11. Scripts: components, restaurant script.

---
*Sources: Rich & Knight "Artificial Intelligence"; Russell & Norvig "AIMA"; Schank & Abelson (1977) on CD and scripts; university lecture notes on predicate-logic resolution, unification and Skolemization (cross-checked).*
