# Theory of Computation (MCC-2.3.3) — Unit IV Notes
**Ravenshaw University, PG MCA (CBCS), 3rd Sem**
**Text:** Hopcroft, Motwani, Ullman — *Introduction to Automata Theory, Languages and Computation* (Ch. 8–9)

**Unit IV syllabus:** The Turing machine · Programming techniques for Turing machines · Extensions to the basic Turing machine · Restricted Turing machines · Turing machines and computers · Non-recursively enumerable languages · Undecidable problem that is recursively enumerable · Undecidable problems about Turing machines · Post's Correspondence Problem · Other undecidable problems

---

## 1. The Turing Machine (TM)

### 1.1 Idea
Proposed by **Alan Turing (1936)** as a model of what "computable" means. A TM has a **finite control**, and an **infinite tape** divided into cells, with a **read/write head** that can move **left or right**.

```
 Tape: ... B | B | 0 | 1 | 1 | 0 | B | B ...   (infinite both directions, blank B)
                        ^
                     [ head ]
                  [ Finite control: state q ]
```

- Initially: input string on the tape (rest blank), head on the **leftmost input symbol**, control in **start state q₀**.
- Each move: based on (current state, scanned symbol) → **write a symbol, change state, move L or R**.
- Unlike FA/PDA, the TM can **rewrite** the tape and **revisit** cells (unbounded, flexible memory).

### 1.2 Formal Definition
**M = (Q, Σ, Γ, δ, q₀, B, F)**

| Symbol | Meaning |
|---|---|
| Q | finite set of states |
| Σ | input alphabet (not containing B) |
| Γ | **tape alphabet**, Σ ⊂ Γ |
| δ | transition function: **Q × Γ → Q × Γ × {L, R}** (may be partial) |
| q₀ | start state |
| B | **blank symbol**, B ∈ Γ − Σ |
| F | set of **final (accepting) states** |

δ(q, X) = (p, Y, D): in state q scanning X → go to state p, write Y over X, move head in direction D.

### 1.3 Instantaneous Description (ID)
Written **X₁X₂…Xᵢ₋₁ q XᵢXᵢ₊₁…Xₙ**: state q, head scanning Xᵢ (the symbol right after q). Move relation ⊢, closure ⊢\*.

### 1.4 Language and Halting
- **L(M) = { w ∈ Σ\* | q₀w ⊢\* αpβ, p ∈ F }** (accepts if it enters a final state).
- By convention, a TM **halts** when it has no move; accepting states typically have no further moves.
- For an input, the TM can: **accept** (halt in final state), **reject** (halt in non-final state / no move), or **loop forever**.

### 1.5 Example: TM for { 0ⁿ1ⁿ | n ≥ 1 }
Strategy: mark a leftmost 0 as X, move right, mark the matching leftmost 1 as Y, return, repeat. Accept if all 0s and 1s are matched.

| State | 0 | 1 | X | Y | B |
|---|---|---|---|---|---|
| →q₀ | (q₁, X, R) | — | — | (q₃, Y, R) | — |
| q₁ | (q₁, 0, R) | (q₂, Y, L) | — | (q₁, Y, R) | — |
| q₂ | (q₂, 0, L) | — | (q₀, X, R) | (q₂, Y, L) | — |
| q₃ | — | — | — | (q₃, Y, R) | (q₄, B, R) |
| \*q₄ | — | — | — | — | — |

Trace on 0011: q₀0011 ⊢ Xq₁011 ⊢ X0q₁11 ⊢ Xq₂0Y1 ⊢ q₂X0Y1 ⊢ Xq₀0Y1 ⊢ XXq₁Y1 ⊢ XXYq₁1 ⊢ XXq₂YY ⊢ Xq₂XYY ⊢ XXq₀YY ⊢ XXYq₃Y ⊢ XXYYq₃B ⊢ XXYYBq₄ → **accepted**.

### 1.6 TM as a Function Computer
Input on tape, **final tape contents = output** (e.g., unary addition, copying, subtraction). Functions computable this way are **(partial) recursive functions**.

### 1.7 Recursive vs Recursively Enumerable Languages
| Class | Definition | TM behaviour |
|---|---|---|
| **Recursively enumerable (RE)** (Turing-recognizable) | L = L(M) for some TM M | halts & accepts if w ∈ L; **may loop forever** if w ∉ L |
| **Recursive (REC)** (decidable) | L = L(M) for a TM that **halts on every input** | always halts: accepts if w ∈ L, rejects if w ∉ L (an **algorithm**) |

Hierarchy: **Regular ⊂ DCFL ⊂ CFL ⊂ Recursive ⊂ RE ⊂ all languages**.

### 1.8 Church–Turing Thesis
"Anything computable by an **algorithm** (effective procedure) can be computed by a Turing machine." A *thesis*, not a theorem (cannot be proved, as "algorithm" is informal). Every other model proposed (λ-calculus, recursive functions, RAM, Post systems) is equivalent to TMs.

---

## 2. Programming Techniques for Turing Machines

Ways to design TMs more easily (none increases power):

1. **Storage in the state (finite memory in control):** state is a tuple [q, data]; e.g., remember the first symbol read to compare with the last (used for languages like 01\* + 10\*).
2. **Multiple tracks:** tape alphabet consists of tuples, so each cell has several "tracks" (e.g., track 1 = input, track 2 = marks ✓); useful for **checking off** symbols, copying, testing primes/divisibility.
3. **Checking off symbols:** mark processed symbols (X, Y above) to count/match.
4. **Shifting over:** move a block of symbols right by k cells to make room (remember displaced symbols in the state).
5. **Subroutines:** a TM fragment with its own start state and a "return" state; the calling TM transfers control to the start state and continues from the return state afterwards (e.g., a **copy** subroutine used inside **multiplication**).

---

## 3. Extensions to the Basic Turing Machine

**Key result:** extensions make TMs **easier to program**, but add **no computational power** (same class of languages).

### 3.1 Multitape TM
- k tapes, each with its **own head**; one move: depends on state and the **k scanned symbols**; writes on each tape, moves each head independently (L, R or S).
- **Theorem:** every language accepted by a multitape TM is accepted by a **single-tape TM**.
- *Construction:* encode k tapes on one tape using **2k tracks** (k for contents, k for head-position markers).
- **Cost:** a multitape TM running in time **T(n)** is simulated by a single-tape TM in time **O(T(n)²)** (quadratic slowdown).

### 3.2 Non-deterministic TM (NTM)
- δ(q, X) is a **set** of possible moves; accepts if **some** sequence of choices reaches a final state.
- **Theorem:** every NTM can be simulated by a deterministic TM (same languages).
- *Construction:* deterministic **multitape TM** performs **breadth-first search** over the tree of ID sequences (tape 1 queue of IDs).
- **Cost:** exponential slowdown (this gap is the root of the **P vs NP** question).

### 3.3 Other Equivalent Variants
Two-way/one-way infinite tape, multidimensional tape, multiple heads on one tape, TM with "stay" move, offline TM — all equivalent.

---

## 4. Restricted Turing Machines

Restrictions that **still** keep full TM power (or classify power precisely):

### 4.1 TM with Semi-infinite Tape
- Tape infinite only to the **right**; the head never moves left of its starting cell (use an end marker/two-track folding).
- **Theorem:** accepts exactly the same languages as ordinary TM.

### 4.2 Multistack Machines
- Like a PDA with **k ≥ 1 stacks** (deterministic variant here).
- One stack = PDA (CFLs only). **Two stacks ⇒ full TM power**: one stack holds tape to the left of the head, the other the tape to the right. **Every RE language is accepted by a two-stack machine.**

### 4.3 Counter Machines
- Counter = stack with only **one stack symbol** (plus bottom marker); can add 1, subtract 1, and test for zero.
- **Every RE language is accepted by a three-counter machine** (via simulation with two stacks, whose contents are encoded as integers).
- **Every RE language is accepted by a two-counter machine** (Minsky): encode three counts as 2ᵃ3ᵇ5ᶜ in one counter.
- **One-counter machine** accepts only a subset of CFLs; **zero-counter** = FA.

| Machine | Power |
|---|---|
| Finite automaton (0 stacks) | Regular |
| PDA (1 stack) | CFL |
| One counter | subset of CFL |
| 2 stacks / 2 counters (non-deterministic or deterministic) | **RE (= TM)** |

---

## 5. Turing Machines and Computers

- **Computers can simulate TMs:** a program can simulate the TM's finite control and tape (tape can be stored on disks; assume unlimited storage is available). So anything a TM computes, a computer can.
- **TMs can simulate computers:** model the computer's memory (address–value pairs) and registers on TM tapes; each instruction cycle is carried out by TM subroutines (search address, read/write, arithmetic). So anything a computer computes, a TM can (**same computability**).
- **Running time:** the TM simulation of a computer is **polynomial-time slower** (about O(n³) steps on a multitape TM, hence roughly O(n⁶) on a single-tape TM, for n computer steps — per HMU's analysis). Hence **polynomial-time on computers ⇔ polynomial-time on TMs** (the basis for the class P).
- Conclusion: TM is a **realistic model for computability** and a reasonable model for **complexity** (up to polynomial factors).

---

## 6. Non-Recursively Enumerable Languages

### 6.1 Closure and Relationship Facts
- Recursive languages are closed under **union, intersection, concatenation, Kleene star, complement**.
- RE languages are closed under union, intersection, concatenation, star, but **NOT complement**.
- **Theorem:** if **L and its complement L̄ are both RE**, then L is **recursive** (run both TMs in parallel; exactly one accepts).
- ⇒ If L is RE but not recursive, then L̄ is **not RE**.

### 6.2 Encoding Turing Machines
Every TM can be encoded as a **binary string** (e.g., using 0ⁱ and 1 separators for states, symbols, directions and transitions). So TMs can be **enumerated**: M₁, M₂, M₃, … ↔ strings w₁, w₂, w₃, … (the i-th string wᵢ encodes Mᵢ; invalid codes = TM accepting ∅).
Since TMs are countable (ℵ₀) and languages are uncountable, **most languages are not RE** (not even describable by any TM).

### 6.3 The Diagonalization Language Ld
**L_d = { wᵢ | wᵢ ∉ L(Mᵢ) }** (strings whose TM does not accept its own code).

**Theorem: L_d is not recursively enumerable.**
*Proof (diagonalization):* build a table with entry (i, j) = 1 if Mᵢ accepts wⱼ. L_d takes the **complement of the diagonal**. If L_d = L(Mₖ) for some k, then wₖ ∈ L_d ⇔ wₖ ∉ L(Mₖ) = L_d, a contradiction.

---

## 7. An Undecidable Problem that is RE — The Universal Language

### 7.1 Universal Turing Machine (UTM)
A TM **U** that takes input **⟨M, w⟩** (code of M plus string w) and **simulates M on w**. It is a "stored-program computer": one fixed machine that runs any TM.

### 7.2 Universal Language
**L_u = { ⟨M, w⟩ | M accepts w }** (the **acceptance problem**, A_TM)

**Theorem:** L_u is **RE** but **not recursive** (undecidable).
- **RE:** U simulates M on w; accepts if M accepts.
- **Not recursive:** if L_u were recursive, its complement would be recursive, and then L_d would be recursive (convert wᵢ to ⟨Mᵢ, wᵢ⟩, run the decider and flip the answer), contradicting that L_d is not even RE.
- Hence **complement of L_u is not RE.**

### 7.3 Halting Problem
**HALT = { ⟨M, w⟩ | M halts on w }** — **RE but undecidable.**
**Proof sketch (contradiction):** suppose a TM H decides HALT. Build D(⟨M⟩): run H on ⟨M, ⟨M⟩⟩; if H says "halts" then **loop forever**, else **halt**. Run D on its own code: D(⟨D⟩) halts ⇔ D(⟨D⟩) does not halt. Contradiction.

### 7.4 Reduction Technique
To prove problem **B undecidable**: show that a known undecidable problem **A reduces to B** (an algorithm converting any instance of A into an instance of B with the same yes/no answer). Then a decider for B would give a decider for A — impossible.

**Summary diagram:**
```
 Recursive (decidable)  ⊂  RE  ⊂  All languages
        ↑ e.g. 0ⁿ1ⁿ, CFLs     ↑ L_u, HALT          ↑ L_d, complement of L_u
```
| Language | Recursive? | RE? |
|---|---|---|
| L_d | No | **No** |
| L_u | No | Yes |
| complement of L_u | No | **No** |
| Any CFL / regular language | Yes | Yes |

---

## 8. Undecidable Problems About Turing Machines

### 8.1 Emptiness Languages
- **L_e = { ⟨M⟩ | L(M) = ∅ }** — **not RE** (complement L_ne is RE but not recursive).
- **L_ne = { ⟨M⟩ | L(M) ≠ ∅ }** — **RE** (guess an accepted string by dovetailing) but **not recursive**.
- So "Does a given TM accept any string?" is **undecidable**.

### 8.2 Rice's Theorem
**A property of the RE languages** is a set of RE languages. It is **trivial** if it holds for **all** RE languages or for **none**.

> **Rice's Theorem:** **Every non-trivial property of RE languages is undecidable.** (No algorithm can decide, given a TM M, whether L(M) has that property.)

**Consequences (all undecidable for a given TM M):**
- Is L(M) empty? finite? infinite?
- Is L(M) regular? context-free? recursive?
- Does L(M) contain a given string (e.g., ε) or all strings (L(M) = Σ\*)?
- Is L(M) = L(M′) (equivalence of two TMs)?

**Caution:** Rice's theorem applies to properties of the **language** L(M), **not** of the machine itself. E.g., "M has more than 5 states" or "M makes at least 10 moves on input w" are **decidable**. Properties that differ between two TMs accepting the *same* language are not covered.

**Proof idea:** reduce L_u to the property: given ⟨M, w⟩, build M′ that simulates M on w and, if M accepts, then behaves like a TM with a language having the property (otherwise accepts nothing). Then L(M′) has the property ⇔ M accepts w.

### 8.3 Problems About TM Specifications (HMU 9.3.4)
Questions about the **structure** of a TM can often be decided (e.g., "Does M have an even number of states?"), unlike questions about its language.

---

## 9. Post's Correspondence Problem (PCP)

### 9.1 Definition
**Instance:** two lists of strings over Σ of equal length k:
**A = w₁, w₂, …, w_k** and **B = x₁, x₂, …, x_k**.
**Question:** is there a sequence of indices **i₁, i₂, …, i_m (m ≥ 1, repetitions allowed)** with
**w_{i₁} w_{i₂} … w_{i_m} = x_{i₁} x_{i₂} … x_{i_m}**?
(Pictured as dominoes with top string wᵢ and bottom string xᵢ.)

### 9.2 Example (has a solution)
| i | List A (wᵢ) | List B (xᵢ) |
|---|---|---|
| 1 | 1 | 111 |
| 2 | 10111 | 10 |
| 3 | 10 | 0 |

Solution: **i = 2, 1, 1, 3**
- A: 10111 · 1 · 1 · 10 = **101111110**
- B: 10 · 111 · 111 · 0 = **101111110** ✓

**Example with no solution:** A = (ab, ba), B = (a, b). Every top string is longer than its bottom string, so the top concatenation is always longer than the bottom one. Quick no-solution check: if \|wᵢ\| > \|xᵢ\| for all i (or < for all i), there is no solution.

### 9.3 Modified PCP (MPCP)
Same as PCP but the **solution must begin with pair 1** (i₁ = 1).

### 9.4 Undecidability
- **Theorem: PCP is undecidable.**
- Proof outline: **L_u ≤ MPCP ≤ PCP** (reductions). Construct a PCP instance from ⟨M, w⟩ whose partial matches simulate the sequence of IDs of M on w; a full match exists ⇔ M accepts w.
- PCP is **RE** (try all index sequences in increasing length) but not recursive.
- Decidable special cases exist (e.g., one-letter alphabet; or k ≤ 2).

### 9.5 Importance
PCP is a convenient starting point for undecidability proofs in **language theory**, since it is simple and string-based (no TMs needed).

---

## 10. Other Undecidable Problems

1. **Halting problem** (does program P halt on input x?) — foundation of undecidability in software.
2. **Ambiguity of a CFG** — via reduction from PCP.
3. **Equivalence of two CFGs; L(G) = Σ\*; L(G₁) ∩ L(G₂) = ∅ (given two CFGs)**; is L(G₁) ⊆ L(G₂)?
4. Is the complement of a CFL also a CFL? Is a CFL regular? (given only a CFG)
5. **Is a given CFL inherently ambiguous?**
6. **Program verification:** does a program have no bugs? Do two programs compute the same function? Will a program print "hello"? (all via Rice-type arguments).
7. **Hilbert's Tenth Problem:** does a polynomial Diophantine equation with integer coefficients have an integer solution? (shown undecidable by Matiyasevich, 1970).
8. **Entscheidungsproblem:** validity of first-order logic sentences is undecidable (Church–Turing, 1936).
9. **Totality:** does a TM halt on **every** input? (not even RE)
10. **Tiling problem** (can a set of Wang tiles tile the plane?).

---

## Quick Revision

- **TM** = (Q, Σ, Γ, δ, q₀, B, F), δ: Q×Γ → Q×Γ×{L,R}; accepts by entering a final state.
- **RE** = accepted by a TM (may loop); **Recursive** = decided by a TM that always halts.
- **Extensions don't add power:** multitape (O(T²) slowdown), NTM (exponential slowdown), semi-infinite tape, **two stacks / two counters = TM**.
- **TM ≡ real computer** in computability (polynomial slowdown in time).
- **L_d: not RE. L_u / HALT: RE, not recursive. Complement of L_u: not RE.**
- **If L and L̄ are RE ⇒ L is recursive.**
- **Rice's theorem:** every non-trivial property of RE languages is undecidable.
- **PCP undecidable**, used to show CFG ambiguity, equivalence etc. undecidable.

## Frequently Asked Exam Questions
1. Define a Turing machine; design TMs for 0ⁿ1ⁿ, wwᴿ, aⁿbⁿcⁿ, binary addition, or unary increment; show IDs.
2. Explain programming techniques: storage in state, multiple tracks, subroutines, shifting over.
3. Prove that a multitape TM is equivalent to a single-tape TM; explain the non-deterministic TM.
4. Explain restricted TMs: semi-infinite tape, multistack, counter machines.
5. Show how a TM can simulate a computer and vice versa.
6. Differentiate recursive and recursively enumerable languages; show that L_d is not RE.
7. Define the universal language and universal TM; prove L_u is undecidable.
8. State and explain Rice's theorem with examples.
9. Define PCP; find a solution for a given instance; sketch why PCP is undecidable.
10. Prove the halting problem is undecidable; list other undecidable problems.

---
*Sources: syllabus PDF (p. 5–6); Hopcroft–Motwani–Ullman Ch. 8–9 (section structure confirmed from the book's contents list and course outlines at Padua, Bolzano, UNH, HMC); standard theory-of-computation material.*
