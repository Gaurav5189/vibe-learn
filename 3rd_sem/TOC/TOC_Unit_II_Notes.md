# Theory of Computation (MCC-2.3.3) — Unit II Notes
**Ravenshaw University, PG MCA (CBCS), 3rd Sem**
**Text:** Hopcroft, Motwani, Ullman — *Introduction to Automata Theory, Languages and Computation* (Ch. 3–4)

**Unit II syllabus:** Regular expressions · Finite automata and regular expressions · Applications of regular expressions · Algebraic laws of regular expressions · Pumping lemma and its application for regular languages · Closure and decision properties of regular languages

---

## 1. Regular Expressions (RE)

- A **regular expression** is an algebraic notation that describes a **regular language**.
- RE and FA are equivalent in power: **a language is regular ⇔ it is described by an RE ⇔ it is accepted by a DFA/NFA/ε-NFA.**

### 1.1 Recursive Definition
**Basis:**
- ε is an RE; L(ε) = {ε}
- ∅ is an RE; L(∅) = ∅
- For any symbol a ∈ Σ, **a** is an RE; L(a) = {a}

**Induction:** if R and S are REs:
| Operation | RE | Language |
|---|---|---|
| Union | R + S (also R \| S) | L(R) ∪ L(S) |
| Concatenation | RS | L(R)L(S) |
| Kleene star | R\* | (L(R))\* |
| Grouping | (R) | L(R) |

### 1.2 Precedence (highest → lowest)
1. **Star (\*)** 
2. **Concatenation (·)** 
3. **Union (+)**

Example: `01* + 1` = `(0(1*)) + 1`, not `(01)* + 1`. Use parentheses to override.

### 1.3 Examples (Σ = {0,1})
| Language | RE |
|---|---|
| All strings | (0+1)\* |
| Strings ending in 01 | (0+1)\*01 |
| Strings starting with 1 | 1(0+1)\* |
| Contains substring 00 | (0+1)\*00(0+1)\* |
| Even-length strings | ((0+1)(0+1))\* |
| Alternating 0s and 1s | (01)\*(ε+0) + (10)\*(ε+1) |
| At least one 1 | 0\*1(0+1)\* |

### 1.4 Shorthand
- **R⁺** = RR\* (one or more)
- **R?** = ε + R (zero or one)
- **Rᵏ** = R repeated k times

---

## 2. Finite Automata and Regular Expressions

**Theorem (Kleene):** Languages accepted by FA = languages described by RE.

### 2.1 FA → RE
**Method A: State Elimination**
1. Convert the FA to one with a **single start state (no incoming edges)** and **single final state (no outgoing edges)**.
2. Label edges with REs (merge parallel edges using +).
3. **Eliminate intermediate states one by one.** To remove state s with incoming edge labeled R₁ (p→s), self-loop S, outgoing R₂ (s→q), add edge p→q labeled **R₁ S\* R₂** (union with any existing p→q label).
4. Final single edge between start and final state = RE.

**Method B: Arden's Theorem** (commonly used in exams)
- If **P** does not contain ε, then the equation **R = Q + RP** has the unique solution **R = QP\***.
- Steps: write an equation for each state (incoming transitions + ε for the start state), substitute and apply Arden's theorem repeatedly; RE of final state(s) is the answer.
- Example: R = ε + R·0 gives Q = ε, P = 0, so R = 0\*.

### 2.2 RE → ε-NFA (Thompson's Construction)
Build the automaton **inductively** by structure of the RE. Each piece has **one start state and one accepting state**.

| RE | Automaton |
|---|---|
| ∅ | start and accept states, no transition |
| ε | start –ε→ accept |
| a | start –a→ accept |
| R + S | new start with ε-moves to start of R and S; ε-moves from accepts of R and S to new accept |
| RS | ε-move from accept of R to start of S |
| R\* | new start/accept; ε-moves: new start→start of R, accept of R→start of R (loop), accept of R→new accept, new start→new accept (skip) |

Then ε-NFA → NFA → DFA (subset construction, Unit I) if needed.

**Summary of conversions:**
```
RE ⇄ ε-NFA ⇄ NFA ⇄ DFA   (all describe regular languages)
```

---

## 3. Applications of Regular Expressions

1. **Text search & pattern matching** — UNIX tools `grep`, `egrep`, `sed`, `awk`; editor find/replace.
2. **Lexical analysis (compilers)** — token patterns for identifiers, numbers, keywords (e.g., identifier = letter(letter+digit)\*); tools like **Lex/Flex** convert REs → NFA → DFA.
3. **Input validation** — email, phone number, PIN code, dates, passwords in forms.
4. **Web & data processing** — URL routing, log file analysis, web scraping, data cleaning.
5. **Protocol/command parsing** and file name matching (wildcards).
6. **Bioinformatics** — searching DNA/protein sequence motifs.

**UNIX-style notations** (extensions, same power): `.` any character, `[a-z]` character class, `[^a]` negation, `+` one or more, `?` optional, `{n}` repetition, `^` start, `$` end.

> Note: Many programming-language "regex" engines (e.g., backreferences) go **beyond** regular languages.

---

## 4. Algebraic Laws of Regular Expressions

Two REs R, S are **equivalent** (R = S) if L(R) = L(S).

### 4.1 Union (+)
- **Commutative:** R + S = S + R
- **Associative:** (R + S) + T = R + (S + T)
- **Identity:** R + ∅ = R
- **Idempotent:** R + R = R

### 4.2 Concatenation
- **Associative:** (RS)T = R(ST)
- **Not commutative:** RS ≠ SR in general
- **Identity:** εR = Rε = R
- **Annihilator:** ∅R = R∅ = ∅

### 4.3 Distributive
- R(S + T) = RS + RT (left)
- (S + T)R = SR + TR (right)

### 4.4 Kleene Star Laws
- (R\*)\* = R\*
- ∅\* = ε
- ε\* = ε
- R⁺ = RR\* = R\*R
- R\* = ε + R⁺
- **(R + ε)\* = R\***
- **(R + S)\* = (R\*S\*)\* = (R\*+S)\* = (R+S\*)\***
- R\*R\* = R\*
- (RS)\*R = R(SR)\*

### 4.5 Proving a Law
Convert both sides to the language they denote and show equality as sets, or use known laws to transform one side into the other.

**Testing a proposed law (HMU method):** replace each variable by a distinct symbol (R→a, S→b) and check whether the resulting expressions describe the same language. Works for the laws above; fails for something like R\* = ... where concrete languages matter.

---

## 5. Pumping Lemma for Regular Languages

### 5.1 Purpose
A tool to prove that a language is **NOT regular**. Idea: an FA has finite memory (n states), so any sufficiently long string must **revisit a state** (pigeonhole principle), creating a loop that can be repeated ("pumped").

### 5.2 Statement
If L is a regular language, then **there exists a constant n** (pumping length, ≤ number of DFA states) such that **every string w ∈ L with |w| ≥ n** can be split as **w = xyz** satisfying:
1. **y ≠ ε** (|y| ≥ 1)
2. **|xy| ≤ n**
3. **xyᵏz ∈ L for all k ≥ 0** (pump down k = 0, or up k ≥ 2)

### 5.3 Proof Sketch
- Let DFA have n states. Take w with |w| ≥ n; reading the first n symbols visits n+1 states → some state repeats.
- The portion between repeats is y (a loop). x = before loop, z = rest.
- Looping y any number of times (or skipping it) still ends in the same final state ⇒ xyᵏz ∈ L.

### 5.4 How to Apply (Proof by Contradiction)
1. **Assume** L is regular, let n be the pumping length (given by the adversary).
2. **Choose** a string w ∈ L with |w| ≥ n (your clever choice, depends on n).
3. **Consider every split** w = xyz with |xy| ≤ n and y ≠ ε (adversary chooses split).
4. **Find a k** (you choose) such that xyᵏz ∉ L.
5. **Contradiction** ⇒ L is **not regular**.

### 5.5 Examples
**(a) L = { 0ᵐ1ᵐ | m ≥ 0 }** 
- Take w = 0ⁿ1ⁿ. Since |xy| ≤ n, y consists only of 0s: y = 0ᵗ, t ≥ 1.
- Pump up (k = 2): xy²z = 0ⁿ⁺ᵗ1ⁿ — more 0s than 1s ∉ L. Contradiction ⇒ **not regular**.

**(b) L = { w ∈ {a,b}\* | w has equal number of a's and b's }**
- Take w = aⁿbⁿ; y is within the a's; pumping changes the a-count only ⇒ ∉ L ⇒ **not regular**.

**(c) L = { 0ⁿ² | n ≥ 1 } (perfect-square length)** 
- Take w = 0ⁿ², y = 0ᵗ, 1 ≤ t ≤ n. Then |xy²z| = n² + t, and n² < n² + t ≤ n² + n < (n+1)² ⇒ not a perfect square ⇒ **not regular**.

**(d) L = { ww | w ∈ {0,1}\* }** — not regular (choose w = 0ⁿ10ⁿ1).

### 5.6 Important Caveats
- The lemma is a **necessary** condition only: **regular ⇒ pumpable**.
- It **cannot prove a language is regular.** A non-regular language may still satisfy the pumping property.
- If a language fails the lemma ⇒ definitely not regular.
- Typical pitfalls: choosing w independent of n; ignoring the |xy| ≤ n condition.
- Can be combined with **closure properties** to prove non-regularity.

---

## 6. Closure Properties of Regular Languages

**Closure** means: applying the operation to regular languages always gives a regular language. If L and M are regular, then these are also regular:

| Operation | Why regular (proof idea) |
|---|---|
| **Union** L ∪ M | RE: R + S; or ε-NFA with new start |
| **Concatenation** LM | RE: RS; or ε-link accept of L's FA to start of M's |
| **Kleene star** L\* | RE: R\*; Thompson construction |
| **Complement** Σ\* − L | Take a **complete DFA**, swap final and non-final states |
| **Intersection** L ∩ M | **Product construction** (pair states, accept if both accept); or De Morgan: (Lᶜ ∪ Mᶜ)ᶜ |
| **Difference** L − M | L ∩ Mᶜ |
| **Reversal** Lᴿ | Reverse all arrows, swap start ↔ final (use new start with ε-moves to old finals if multiple) |
| **Homomorphism** h(L) | Substitute h(a) for each symbol a in the RE |
| **Inverse homomorphism** h⁻¹(L) | In the DFA, define δ′(q, a) = δ̂(q, h(a)) |
| **Prefix / Quotient** | Modify final states of DFA (states from which a final state is reachable) |

**Homomorphism:** a function h: Σ → Γ\* extended to strings by h(a₁…aₖ) = h(a₁)…h(aₖ). Example: h(0) = ab, h(1) = ε.

**Product construction (intersection):** states (p, q), start (p₀, q₀), δ((p,q),a) = (δ₁(p,a), δ₂(q,a)), final = F₁ × F₂.

**Use of closure properties:** to prove a language L is **not regular**, show that combining L with a regular language via a closure operation yields a known non-regular language. Example: if L = {w | #0 = #1} were regular, L ∩ 0\*1\* = {0ⁿ1ⁿ} would be regular — contradiction.

---

## 7. Decision Properties of Regular Languages

A **decision property** is a question about a language that an **algorithm** can always answer (yes/no). A regular language may be given as a DFA, NFA, ε-NFA or RE (convert to DFA first; note NFA → DFA can cost up to 2ⁿ states).

| Question | Algorithm |
|---|---|
| **Emptiness:** is L = ∅? | Graph search: is **no** final state reachable from the start state? |
| **Membership:** is w ∈ L? | Simulate DFA on w: O(\|w\|) steps; accept if it ends in a final state |
| **Finiteness / Infiniteness** | L is **infinite** iff the DFA (after removing unreachable and dead states) has a **cycle** on some path from start to a final state; equivalently, accepts a string with n ≤ \|w\| < 2n |
| **Equivalence:** L = M? | **Table-filling (distinguishability) algorithm** on the union of both DFAs: L = M iff start states are **equivalent**. Alternative: check (L − M) ∪ (M − L) = ∅ |
| **Containment:** L ⊆ M? | Check L ∩ Mᶜ = ∅ (emptiness test) |
| **Universality:** L = Σ\*? | Check complement Lᶜ = ∅ |

### 7.1 Table-Filling Algorithm (Equivalence / Minimization)
1. List all pairs of states {p, q}.
2. **Mark** pair {p, q} as **distinguishable** if one is final and the other non-final (base case).
3. Repeat: mark {p, q} if for some symbol a, {δ(p,a), δ(q,a)} is already marked.
4. Stop when no new marks. **Unmarked pairs = equivalent states** (can be merged).

### 7.2 DFA Minimization
- Remove unreachable states, then merge equivalent states found by table-filling.
- Result is the **unique minimum-state DFA** for the language (up to renaming).
- Two DFAs are equivalent iff their minimal DFAs are identical (isomorphic).

---

## Quick Revision

- **RE operators:** + (union), concatenation, \* (star); precedence: \* > concat > +.
- **RE ⇔ FA:** RE → ε-NFA (Thompson); FA → RE (state elimination / Arden: R = Q + RP ⇒ R = QP\*).
- **Applications:** grep, lexical analysis (Lex), validation, text search.
- **Pumping lemma:** w = xyz, y ≠ ε, |xy| ≤ n, xyᵏz ∈ L ∀k ≥ 0. Used **only to prove non-regularity**.
- **Closed under:** union, concatenation, star, complement, intersection, difference, reversal, homomorphism, inverse homomorphism.
- **Decidable:** emptiness, membership, finiteness, equivalence, containment (via table-filling / product construction).

## Frequently Asked Exam Questions
1. Define regular expression. Write REs for given languages (ends with 01, contains 00, etc.).
2. Convert a given RE to ε-NFA / DFA.
3. Convert a given DFA to RE using Arden's theorem / state elimination.
4. State and prove the pumping lemma. Show that {0ⁿ1ⁿ} (or {ww}, primes, squares) is not regular.
5. List and prove closure properties of regular languages (complement, intersection, reversal).
6. Explain decision properties; give an algorithm for emptiness/equivalence of regular languages.
7. Minimize a given DFA (table-filling).
8. Write algebraic laws of regular expressions.

---
*Sources: syllabus PDF (p. 5–6); Hopcroft–Motwani–Ullman Ch. 3–4 (section structure confirmed via course schedules and lecture notes from Chalmers TMV027, Cornell CS3810 and University of Bolzano); standard theory-of-computation material.*
