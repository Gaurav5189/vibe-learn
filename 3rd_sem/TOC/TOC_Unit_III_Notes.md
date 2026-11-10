# Theory of Computation (MCC-2.3.3) — Unit III Notes
**Ravenshaw University, PG MCA (CBCS), 3rd Sem**
**Text:** Hopcroft, Motwani, Ullman — *Introduction to Automata Theory, Languages and Computation* (Ch. 5–7)

**Unit III syllabus:** Context-Free Grammars · Parse trees · Ambiguity in grammars and languages · Pushdown automata · The language of a PDA · Equivalence of PDAs and CFGs · Deterministic pushdown automata · Chomsky Normal Form · Pumping Lemma for CFLs · Decision properties of CFLs

---

## 1. Context-Free Grammars (CFG)

### 1.1 Why CFG?
Regular languages cannot handle **nested/recursive structure** (balanced brackets, 0ⁿ1ⁿ, arithmetic expressions). CFGs can. CFGs are the basis for **programming language syntax and parsers**.

### 1.2 Definition
A CFG is a **4-tuple G = (V, T, P, S)**:

| Symbol | Meaning |
|---|---|
| V | finite set of **variables** (non-terminals) |
| T | finite set of **terminals** (V ∩ T = ∅) |
| P | finite set of **productions** of the form A → α, where A ∈ V, α ∈ (V ∪ T)\* |
| S | **start symbol**, S ∈ V |

### 1.3 Derivations
- **Derivation step (⇒):** replace a variable by the body of one of its productions. ⇒\* means zero or more steps.
- **Sentential form:** any string of variables/terminals derivable from S.
- **Leftmost derivation (⇒lm):** always replace the **leftmost** variable. **Rightmost (⇒rm):** always replace the rightmost variable.
- **Language of G:** L(G) = { w ∈ T\* | S ⇒\* w }. A language is **context-free (CFL)** if some CFG generates it.

### 1.4 Examples
| Language | Grammar |
|---|---|
| { 0ⁿ1ⁿ \| n ≥ 0 } | S → 0S1 \| ε |
| Palindromes over {0,1} | S → ε \| 0 \| 1 \| 0S0 \| 1S1 |
| Balanced parentheses | S → (S)S \| ε |
| Arithmetic expressions | E → E+T \| T; T → T\*F \| F; F → (E) \| id |

**Derivation example** (S → 0S1 | ε), string 0011:
S ⇒ 0S1 ⇒ 00S11 ⇒ 0011

### 1.5 Facts
- Every **regular language is context-free** (CFL ⊃ regular).
- CFGs generate **Type-2** languages in the Chomsky hierarchy.

### 1.6 Applications
Parsers and compilers (YACC/Bison), XML/HTML document type definitions (DTD), programming language syntax (BNF), natural language processing.

---

## 2. Parse Trees

A **parse (derivation) tree** shows the derivation as a tree:
- **Root** = start symbol (S).
- **Interior nodes** = variables; children of A are the symbols of the body of a production A → X₁X₂…Xₖ (left to right).
- **Leaves** = terminals or ε.
- **Yield** = string formed by concatenating leaves left to right.

Example for 0011 with S → 0S1 | ε:
```
        S
     /  |  \
    0   S   1
      / | \
     0  S  1
        |
        ε
```
Yield = 0 0 ε 1 1 = 0011.

**Equivalences (for any CFG, variable A, string w ∈ T\*):** the following are all equivalent:
1. A ⇒\* w (a derivation exists)
2. A ⇒lm\* w (leftmost derivation)
3. A ⇒rm\* w (rightmost derivation)
4. A parse tree with root A and yield w
5. Recursive inference (bottom-up) that w is in the language of A

---

## 3. Ambiguity in Grammars and Languages

### 3.1 Ambiguous Grammar
A CFG is **ambiguous** if **some string** in L(G) has **more than one parse tree** (equivalently, more than one leftmost derivation, or more than one rightmost derivation).

**Example:** E → E + E | E \* E | (E) | id
String `id + id * id` has two parse trees:
- (id + id) \* id → groups addition first
- id + (id \* id) → groups multiplication first

Ambiguity is a problem for compilers: the meaning (evaluation) of the string becomes unclear.
Another classic: the **dangling else** (if-if-else).

### 3.2 Removing Ambiguity
Enforce **precedence** and **associativity** by adding levels (variables):
```
E → E + T | T          (+ lowest precedence, left associative)
T → T * F | F          (* binds tighter)
F → ( E ) | id
```
This grammar is **unambiguous** and generates the same language.

### 3.3 Inherent Ambiguity
- A **language** is **inherently ambiguous** if **every** grammar for it is ambiguous.
- Example: L = { aⁿbⁿcᵐdᵐ | n,m ≥ 1 } ∪ { aⁿbᵐcᵐdⁿ | n,m ≥ 1 }.
- **No general algorithm** exists to decide whether a given CFG is ambiguous, nor to remove ambiguity (undecidable).
- Not every ambiguous grammar can be fixed by rewriting, but most practical programming-language ambiguities can.

---

## 4. Pushdown Automata (PDA)

### 4.1 Idea
A PDA = **ε-NFA + a stack** (unbounded memory used in LIFO manner). The stack lets it count and match nested structure, e.g. 0ⁿ1ⁿ: push for each 0, pop for each 1.

```
Input tape → [ Finite control ] → accept/reject
                    ↕
                 [ Stack ]
```

### 4.2 Formal Definition (Non-deterministic PDA)
**P = (Q, Σ, Γ, δ, q₀, Z₀, F)**

| Symbol | Meaning |
|---|---|
| Q | finite set of states |
| Σ | input alphabet |
| Γ | **stack alphabet** |
| δ | transition function: Q × (Σ ∪ {ε}) × Γ → finite subsets of Q × Γ\* |
| q₀ | start state |
| Z₀ | **initial stack symbol** (stack bottom marker), Z₀ ∈ Γ |
| F | set of accepting states |

**Move meaning:** δ(q, a, X) ∋ (p, γ) = in state q, reading input a (or ε), with X on top of stack: go to state p, **replace X by γ** (top is the leftmost symbol of γ).
- γ = ε → **pop**; γ = X → no change; γ = YX → **push** Y.

### 4.3 Instantaneous Description (ID)
Configuration = **(q, w, γ)**: current state, remaining input, stack contents (top at left).
Move: (q, aw, Xβ) ⊢ (p, w, γβ) if (p, γ) ∈ δ(q, a, X). ⊢\* = zero or more moves.

### 4.4 Example: PDA for { wwᴿ | w ∈ {0,1}\* } (even palindromes)
States q₀ (push), q₁ (match/pop), q₂ (accept).
- δ(q₀, 0, Z₀) = {(q₀, 0Z₀)}, δ(q₀, 1, Z₀) = {(q₀, 1Z₀)}
- δ(q₀, 0, 0) = {(q₀, 00), (q₁, ε)}, δ(q₀, 1, 1) = {(q₀, 11), (q₁, ε)}, mixed: δ(q₀, 0, 1) = {(q₀, 01)}, δ(q₀, 1, 0) = {(q₀, 10)}
- δ(q₀, ε, Z₀) = {(q₂, Z₀)}, δ(q₀, ε, 0) = {(q₁, 0)}, δ(q₀, ε, 1) = {(q₁, 1)} (**guess the middle**)
- δ(q₁, 0, 0) = {(q₁, ε)}, δ(q₁, 1, 1) = {(q₁, ε)}
- δ(q₁, ε, Z₀) = {(q₂, Z₀)}

Non-determinism is needed to **guess the midpoint**.

### 4.4b Example: PDA for { 0ⁿ1ⁿ | n ≥ 1 }
Push a symbol X on each 0; on the first 1, switch state and pop X on each 1; accept when stack returns to Z₀.

---

## 5. The Language of a PDA

Two ways of acceptance:

1. **Acceptance by final state:**
   L(P) = { w | (q₀, w, Z₀) ⊢\* (q, ε, α), q ∈ F } — stack contents don't matter.
2. **Acceptance by empty stack:**
   N(P) = { w | (q₀, w, Z₀) ⊢\* (q, ε, ε) } — stack empty after consuming input; F irrelevant.

**Theorem:** Both methods accept exactly the same class of languages (CFLs).

**Conversions:**
- **Empty stack → final state:** add a new bottom marker X₀ below Z₀ and a new start state p₀ (pushes Z₀ on X₀). When X₀ appears on top (original stack emptied), move to new final state p_f.
- **Final state → empty stack:** add X₀ below Z₀; from every final state, on ε move to a new state p that **pops everything** (including X₀). Also add protection so that an accidental emptying of the original stack doesn't accept falsely (hence the X₀).

---

## 6. Equivalence of PDAs and CFGs

**Theorem:** A language is context-free ⇔ it is accepted by some PDA.

### 6.1 CFG → PDA (empty-stack acceptance)
Given G = (V, T, P, S), build PDA with **one state q**, input alphabet T, stack alphabet V ∪ T, start stack symbol S:
- For each variable A: **δ(q, ε, A) = { (q, β) | A → β ∈ P }** (replace A by a production body)
- For each terminal a: **δ(q, a, a) = { (q, ε) }** (match and pop)

The PDA simulates a **leftmost derivation**; the stack holds the unprocessed part of the sentential form.

**Example:** S → 0S1 | ε. PDA: δ(q, ε, S) = {(q, 0S1), (q, ε)}, δ(q, 0, 0) = {(q, ε)}, δ(q, 1, 1) = {(q, ε)}.

### 6.2 PDA → CFG
Given PDA P (empty-stack acceptance), construct grammar with variables of form **[pXq]** meaning: "starting in state p with X on top, the PDA can pop X and reach state q, consuming the string generated by [pXq]." Plus start symbol S.
- **S → [q₀ Z₀ p]** for every state p
- If δ(q, a, X) contains (r, Y₁Y₂…Yₖ), then for all state sequences r₁,…,rₖ add  
  **[q X rₖ] → a [r Y₁ r₁] [r₁ Y₂ r₂] … [rₖ₋₁ Yₖ rₖ]**
- If (r, ε) ∈ δ(q, a, X): **[q X r] → a**

(Here a may be ε.) The proof is by induction on the number of steps. Grammar may be large (many useless variables).

---

## 7. Deterministic Pushdown Automata (DPDA)

### 7.1 Definition
A PDA is **deterministic** if it has **at most one choice** in every situation:
1. δ(q, a, X) has **at most one** element for all q ∈ Q, a ∈ Σ ∪ {ε}, X ∈ Γ.
2. If δ(q, a, X) ≠ ∅ for some a ∈ Σ, then **δ(q, ε, X) = ∅** (no choice between reading and ε-move).

### 7.2 Language Class
- DPDAs accept by **final state**: languages are **Deterministic CFLs (DCFLs)**.
- **Regular ⊂ DCFL ⊂ CFL** (proper containments).
- DPDA with empty-stack acceptance accepts only **prefix-free** languages (no string is a proper prefix of another), so it is weaker.

### 7.3 Key Differences from NPDA
| Feature | NPDA | DPDA |
|---|---|---|
| Power | all CFLs | DCFLs only |
| Equivalence of final state / empty stack | yes | **no** (final-state is stronger) |
| Example not accepted | — | { wwᴿ } (needs guessing the middle) |
| Example accepted | { wwᴿ } | { wcwᴿ } (marker c shows the middle), { 0ⁿ1ⁿ } |

### 7.4 Properties
- DCFLs are **closed under complement** (but not under union or intersection).
- Every DCFL has an **unambiguous grammar** (but not conversely: some unambiguous CFLs are not DCFLs, e.g. wwᴿ).
- DPDAs matter in practice: deterministic parsers (LL, LR/YACC) use them.

---

## 8. Chomsky Normal Form (CNF)

### 8.1 Definition
A CFG is in **CNF** if every production has the form
- **A → BC** (two variables), or
- **A → a** (one terminal).

**Theorem:** Every CFL without ε is generated by a CFG in CNF.
Parse trees in CNF are **binary trees** (useful for proofs, CYK algorithm, pumping lemma).

### 8.2 Simplification of CFG (do in this order)
1. **Eliminate ε-productions** 
   - Find **nullable** variables (A ⇒\* ε): A is nullable if A → ε, or A → B₁…Bₖ with all Bᵢ nullable.
   - For each production, add versions with every subset of nullable variables removed; delete all A → ε. (ε ∈ L is handled separately.)
2. **Eliminate unit productions (A → B)**
   - Find all **unit pairs** (A, B) with A ⇒\* B using only unit productions; for each pair add A → α for every **non-unit** production B → α; delete unit productions.
3. **Eliminate useless symbols**
   - Keep only **generating** symbols (derive some terminal string), then only **reachable** symbols (reachable from S). Order matters: generating first, then reachable.

### 8.3 Conversion to CNF (after simplification)
1. For each terminal a appearing in a body of length ≥ 2, add a new variable **Cₐ → a** and replace a by Cₐ.
2. Break bodies longer than 2: A → B₁B₂…Bₖ (k ≥ 3) becomes A → B₁D₁, D₁ → B₂D₂, …, Dₖ₋₂ → Bₖ₋₁Bₖ.

**Example:** S → aSb | ab
- Step 1: S → A S B | A B, A → a, B → b
- Step 2: S → A D | A B, D → S B, A → a, B → b ✓ (CNF)

### 8.4 Greibach Normal Form (GNF) — for reference
Every production is **A → aα** (terminal followed by zero or more variables). Used in top-down parsing; every non-ε CFL has a GNF grammar.

---

## 9. Pumping Lemma for Context-Free Languages

### 9.1 Statement
If L is a CFL, then **there exists a constant n** such that every string **z ∈ L with |z| ≥ n** can be written as **z = uvwxy** with:
1. **|vwx| ≤ n**
2. **vx ≠ ε** (at least one of v, x non-empty)
3. **uvⁱwxⁱy ∈ L for all i ≥ 0**

(Two places are pumped **simultaneously**, unlike the regular case.)

### 9.2 Proof Idea
- Take CNF grammar with k variables; set **n = 2ᵏ**. A string of length ≥ n has a parse tree of height > k (binary tree), so some **root-to-leaf path has a repeated variable A**.
- Subtree under the upper A yields vwx; the lower A yields w. So A ⇒\* vAx and A ⇒\* w; repeating the first step i times or skipping it gives uvⁱwxⁱy.
- Choose the lowest repetition so |vwx| ≤ n.

### 9.3 Using It (Proof by Contradiction)
1. Assume L is context-free; let n be the pumping length.
2. Pick a string **z ∈ L, |z| ≥ n** (depends on n).
3. Consider **all** splits z = uvwxy with |vwx| ≤ n, vx ≠ ε.
4. Show for each split there is an **i** with uvⁱwxⁱy ∉ L.
5. Contradiction ⇒ L is **not context-free**.

### 9.4 Examples
**(a) L = { aⁿbⁿcⁿ | n ≥ 1 } is not a CFL.**
- Take z = aⁿbⁿcⁿ. Since |vwx| ≤ n, vwx **cannot contain both a's and c's**. So pumping (v, x) affects at most two of the three symbols, leaving the third count unchanged.
- uv²wx²y has unequal counts ⇒ ∉ L. Contradiction.

**(b) L = { ww | w ∈ {0,1}\* } is not a CFL.** Take z = 0ⁿ1ⁿ0ⁿ1ⁿ; any vwx of length ≤ n lies within a block or straddles two adjacent blocks; pumping breaks the pattern.

**(c) L = { aⁱbʲcᵏ | i < j < k }** is not a CFL (pumping up/down breaks the strict ordering).

### 9.5 Notes
- Necessary condition only: it can prove a language is **not** CFL, never that it **is** CFL.
- Also used with closure properties.

---

## 10. Decision Properties of CFLs

A CFL is given by a **CFG** (or PDA, converted to a CFG). Conversion of a CFG to CNF takes **polynomial time**.

### 10.1 Decidable Problems
| Question | Algorithm |
|---|---|
| **Emptiness** (is L = ∅?) | Compute **generating** symbols; L ≠ ∅ iff the start symbol S is generating |
| **Membership** (is w ∈ L?) | **CYK algorithm** on CNF grammar, **O(n³)** time for \|w\| = n |
| **Finiteness** (is L finite?) | CNF grammar; remove useless symbols (and ε); build graph with edge A → B (or C) for each production A → BC; **L is infinite iff the graph has a cycle** |

**CYK algorithm (idea):** table T[i][j] = set of variables deriving substring of length j starting at position i.
- length 1: A ∈ T[i][1] if A → wᵢ.
- length > 1: A ∈ T[i][j] if A → BC and for some split k, B ∈ T[i][k], C ∈ T[i+k][j−k].
- w ∈ L iff S ∈ T[1][n].

### 10.2 Undecidable Problems (no algorithm exists)
- Is a given CFG **ambiguous**?
- Is L(G₁) = L(G₂) (**equivalence**)? Is L(G₁) ⊆ L(G₂)?
- Is L(G) = Σ\* (**universality**)?
- Is L(G₁) ∩ L(G₂) = ∅?
- Is the CFL **inherently ambiguous**?
- Is L(G) regular? (given only a CFG)

---

## 11. Closure Properties of CFLs (supporting topic)

| Operation | Closed? |
|---|---|
| Union, concatenation, Kleene star | **Yes** |
| Reversal, homomorphism, inverse homomorphism, substitution | **Yes** |
| Intersection with a **regular** language | **Yes** |
| **Intersection** of two CFLs | **No** |
| **Complement** | **No** |
| Difference | No (L − M with M regular is closed) |

**Counterexample (intersection):** L₁ = {aⁿbⁿcᵐ} and L₂ = {aᵐbⁿcⁿ} are CFLs, but L₁ ∩ L₂ = {aⁿbⁿcⁿ} is not.

---

## Quick Revision

- **CFG** = (V, T, P, S); **parse tree** shows derivation; **ambiguous** = ≥ 2 parse trees for one string.
- **PDA** = ε-NFA + stack; (Q, Σ, Γ, δ, q₀, Z₀, F); accepts by **final state** or **empty stack** (equivalent for NPDA).
- **CFG ⇔ PDA:** CFG→PDA by simulating leftmost derivation; PDA→CFG with variables [pXq].
- **DPDA** < NPDA in power; **Regular ⊂ DCFL ⊂ CFL**.
- **CNF:** A → BC | a. Steps: remove ε, unit, useless; then fix terminals & long bodies.
- **Pumping lemma (CFL):** z = uvwxy, |vwx| ≤ n, vx ≠ ε, uvⁱwxⁱy ∈ L.
- **Decidable:** emptiness, membership (CYK), finiteness. **Undecidable:** ambiguity, equivalence, universality.

## Frequently Asked Exam Questions
1. Define CFG. Write CFGs for given languages (palindromes, 0ⁿ1ⁿ, balanced parentheses, aⁿb²ⁿ).
2. What is a parse tree? Explain leftmost/rightmost derivations with an example.
3. What is ambiguity? Show a grammar is ambiguous and remove the ambiguity.
4. Define PDA; design a PDA for 0ⁿ1ⁿ or wwᴿ; show IDs for a string.
5. Explain acceptance by final state vs empty stack; convert between them.
6. Convert a CFG to a PDA (and PDA to a CFG).
7. Differentiate DPDA and NPDA; show that wwᴿ is not accepted by a DPDA.
8. Simplify a CFG and convert it into CNF.
9. State and prove the pumping lemma for CFLs; show {aⁿbⁿcⁿ} / {ww} is not a CFL.
10. Explain decision properties of CFLs; CYK algorithm.

---
*Sources: syllabus PDF (p. 5–6); Hopcroft–Motwani–Ullman Ch. 5–7 (chapter/section structure confirmed from the book's contents list and university course outlines at Padua, Bolzano, IIT Kharagpur, ODU); standard theory-of-computation material.*
