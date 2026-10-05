# Theory of Computation (MCC-2.3.3) — Unit I Notes
**Ravenshaw University, PG MCA (CBCS), 3rd Sem**
**Text:** Hopcroft, Motwani, Ullman — *Introduction to Automata Theory, Languages and Computation* (Ch. 1–2)

**Unit I syllabus:** Introduction to finite automata · Central concepts of automata theory · Informal picture of finite automata · Deterministic finite automata · Non-deterministic finite automata · Application · Formal language

---

## 1. Introduction to Finite Automata

- **Automata theory** = study of abstract computing machines and the problems they can solve.
- **Finite Automaton (FA)** = simplest machine model with a **finite number of states** and **no extra memory** (only its current state).
- It reads an input string symbol by symbol, changes state, and finally **accepts or rejects**.
- Why study it? It models real hardware/software: digital circuits, protocols, text search, compilers (lexical analysis).
- **Hierarchy preview (Chomsky):**

| Language class | Machine |
|---|---|
| Regular | Finite Automaton |
| Context-free | Pushdown Automaton |
| Context-sensitive | Linear Bounded Automaton |
| Recursively enumerable | Turing Machine |

- **Two big questions of the course:** *Computability* (what can be solved at all?) and *Complexity* (what can be solved efficiently?).

---

## 2. Central Concepts of Automata Theory

### 2.1 Alphabet (Σ)
A **finite, non-empty set of symbols**.
- Binary: Σ = {0, 1}
- Lowercase letters: Σ = {a, b, …, z}

### 2.2 String (word)
A **finite sequence of symbols** chosen from an alphabet.
- 01101 is a string over {0, 1}.
- **Empty string ε (epsilon):** string with zero symbols; valid over any alphabet.
- **Length |w|:** number of symbols. |01101| = 5, |ε| = 0.

### 2.3 Powers of an Alphabet
- Σᵏ = set of all strings of length **k** over Σ.
- Σ⁰ = {ε} (for every Σ).
- For Σ = {0,1}: Σ¹ = {0,1}, Σ² = {00,01,10,11}.
- **Σ\*** (Kleene closure) = Σ⁰ ∪ Σ¹ ∪ Σ² ∪ … = all strings over Σ (includes ε).
- **Σ⁺** = Σ¹ ∪ Σ² ∪ … = Σ\* − {ε} (non-empty strings).

### 2.4 Operations on Strings
- **Concatenation:** if x = a₁…aᵢ and y = b₁…bⱼ then xy = a₁…aᵢb₁…bⱼ. Identity: εw = wε = w. |xy| = |x| + |y|.
- **Reversal (wᴿ):** string written backwards.
- **Prefix / Suffix / Substring:** leading / trailing / contiguous part of a string. Example: for "abc", prefixes = ε, a, ab, abc.
- **Power of a string:** wⁿ = w concatenated n times; w⁰ = ε.

### 2.5 Language
A **set of strings, all chosen from some Σ\***. (Any L ⊆ Σ\*.)
- Σ\* is a language over any Σ (infinite).
- **∅** (empty language, no strings) and **{ε}** (only the empty string) are different languages.
- Examples over {0,1}: 
  - L₁ = strings with equal number of 0s and 1s
  - L₂ = binary numbers whose value is prime
  - L₃ = { 0ⁿ1ⁿ | n ≥ 1 }

### 2.6 Set-Former Notation
`{ w | (condition on w) }` — e.g., { w | w has an even number of 0s }. Reads: "set of w such that …".

### 2.7 Problems as Language Membership
In automata theory, a **problem** = deciding whether a given string **w belongs to language L** (membership problem). Every yes/no problem can be encoded this way.

---

## 3. Informal Picture of Finite Automata

Think of an FA as a **machine with a control unit and an input tape**:

```
 Input tape:  | 1 | 0 | 1 | 1 | 0 | ...   (read left→right)
                ^
                |  read head (moves one cell right per step, never writes)
          +-------------+
          | Finite      |---> Accept / Reject
          | Control     |
          | (states)    |
          +-------------+
```

- Starts in the **start state**, reads one symbol at a time.
- On each symbol, **moves to a next state** according to a transition rule.
- After the input ends: if in a **final (accepting) state** → string **accepted**; else **rejected**.
- Memory = only "which state am I in" → hence "finite".

**Example – On/Off switch:** states {off, on}; input "push". off –push→ on, on –push→ off.

**Transition diagram conventions:**
- Circle = state; **arrow with no source** = start state
- **Double circle** = accepting state
- Labeled arrow = transition on that symbol

---

## 4. Deterministic Finite Automata (DFA)

### 4.1 Definition
A DFA is a **5-tuple A = (Q, Σ, δ, q₀, F)**:

| Symbol | Meaning |
|---|---|
| Q | finite set of **states** |
| Σ | finite **input alphabet** |
| δ | **transition function** δ: Q × Σ → Q |
| q₀ | **start state**, q₀ ∈ Q |
| F | set of **final/accepting states**, F ⊆ Q |

**"Deterministic"** = for every state and input symbol there is **exactly one** next state.

### 4.2 Ways to Represent
1. **Transition diagram** (graph)
2. **Transition table** (rows = states, columns = symbols; mark start with →, final with \*)

### 4.3 Extended Transition Function (δ̂)
Extends δ from a symbol to a whole string:
- Basis: δ̂(q, ε) = q
- Induction: δ̂(q, xa) = δ( δ̂(q, x), a )

### 4.4 Language of a DFA
L(A) = { w | δ̂(q₀, w) ∈ F }
A language is **regular** iff some DFA accepts it.

### 4.5 Example
**DFA accepting all binary strings ending in "01"** (Σ = {0,1}):

| State | on 0 | on 1 |
|---|---|---|
| →q₀ | q₁ | q₀ |
| q₁ | q₁ | q₂ |
| \*q₂ | q₁ | q₀ |

- Trace on 1101: q₀ →1 q₀ →1 q₀ →0 q₁ →1 q₂ ∈ F → **accepted**.
- Trace on 110: ends in q₁ ∉ F → **rejected**.

### 4.6 Design Tips
- Decide what each state must "remember" (e.g., last symbols seen, count mod k).
- Every state needs a transition for **every** symbol (add a **dead/trap state** if needed).
- Test with accepted and rejected strings, including ε.

---

## 5. Non-deterministic Finite Automata (NFA)

### 5.1 Idea
Like a DFA but, from a state on a symbol, the machine may have **zero, one, or several** next states. It can be viewed as being **in several states at once** (guessing/parallel choices).

### 5.2 Definition
NFA is a **5-tuple N = (Q, Σ, δ, q₀, F)** — same as DFA except:

**δ: Q × Σ → 2^Q** (power set of Q; i.e., returns a *set* of states, possibly empty).

### 5.3 Extended Transition Function
- δ̂(q, ε) = {q}
- δ̂(q, xa) = ⋃ δ(p, a) over all p ∈ δ̂(q, x)

### 5.4 Language of an NFA
L(N) = { w | δ̂(q₀, w) ∩ F ≠ ∅ }
A string is accepted if **at least one** path leads to a final state.

### 5.5 Example
**NFA accepting strings ending in "01":**

| State | on 0 | on 1 |
|---|---|---|
| →q₀ | {q₀, q₁} | {q₀} |
| q₁ | ∅ | {q₂} |
| \*q₂ | ∅ | ∅ |

Simpler than the DFA — q₀ "guesses" when the final "01" starts.

### 5.6 Equivalence of DFA and NFA
- Every DFA is trivially an NFA.
- **Every NFA has an equivalent DFA** → NFAs accept exactly the **regular languages**; they add convenience, **not power**.

### 5.7 Subset Construction (NFA → DFA)
Given NFA N = (Q_N, Σ, δ_N, q₀, F_N), build DFA D:
1. **States of D** = subsets of Q_N (up to 2ⁿ states).
2. **Start state** = {q₀}.
3. **Transition:** δ_D(S, a) = ⋃ δ_N(p, a) for all p ∈ S.
4. **Final states** = subsets containing at least one state of F_N.
5. Build only **reachable** subsets (lazy evaluation) to avoid blow-up.

**Worst case:** an n-state NFA may need **2ⁿ** DFA states.

### 5.8 NFA with ε-Transitions (ε-NFA)
- Allows moves **without consuming input** (on ε).
- δ: Q × (Σ ∪ {ε}) → 2^Q
- **ε-closure(q)** = set of states reachable from q using only ε-moves (includes q).
- ε-NFAs are still equivalent to DFAs (eliminate ε via ε-closures + subset construction).
- Useful for easily combining automata (union, concatenation, star).

### 5.9 DFA vs NFA

| Feature | DFA | NFA |
|---|---|---|
| Next states | exactly one | zero, one, or many |
| δ | Q × Σ → Q | Q × Σ → 2^Q |
| ε-moves | not allowed | allowed (ε-NFA) |
| Design | harder, larger | easier, smaller |
| Execution | simple, fast | needs backtracking/parallel tracking |
| Power | regular languages | regular languages |

---

## 6. Applications of Finite Automata

1. **Text search / pattern matching** — find words/patterns in text (e.g., `grep`, search in editors); build an NFA for the keywords, convert to DFA, scan text once.
2. **Lexical analysis (compilers)** — scanner breaks source code into tokens (identifiers, keywords, numbers) using regular expressions → NFA → DFA (tools like Lex/Flex).
3. **Digital circuit design & verification** — sequential circuits/state machines.
4. **Protocol and software modelling** — network protocols, communication, vending machine/traffic light/elevator controllers, UI flow.
5. **Validation** — checking email, phone, identifiers, numeric formats.
6. **Game/AI behaviour** — finite state machines for characters.
7. **Model checking** — verifying finite-state systems.

---

## 7. Formal Language

- A **formal language** is a (possibly infinite) set of strings over a finite alphabet, defined by **precise mathematical rules** (no ambiguity, unlike natural languages).
- A language can be described by: a **grammar** (generates strings), an **automaton** (recognizes strings), or a **regular expression**.
- **Operations on languages** (L, L₁, L₂ over Σ):
  - **Union:** L₁ ∪ L₂
  - **Concatenation:** L₁L₂ = { xy | x ∈ L₁, y ∈ L₂ }
  - **Kleene star:** L\* = L⁰ ∪ L¹ ∪ L² ∪ … (L⁰ = {ε})
  - **Positive closure:** L⁺ = L¹ ∪ L² ∪ …
  - **Complement:** Σ\* − L
  - **Intersection:** L₁ ∩ L₂
  - **Reversal:** Lᴿ = { wᴿ | w ∈ L }
- **Chomsky hierarchy** (relation between grammars, languages, machines):

| Type | Grammar | Language | Machine |
|---|---|---|---|
| 3 | Regular | Regular | FA (DFA/NFA) |
| 2 | Context-free | Context-free | Pushdown automaton |
| 1 | Context-sensitive | Context-sensitive | Linear bounded automaton |
| 0 | Unrestricted | Recursively enumerable | Turing machine |

  Each class properly contains the one below it: Type 3 ⊂ Type 2 ⊂ Type 1 ⊂ Type 0.

---

## Quick Revision

- **Alphabet** = finite non-empty set of symbols; **String** = finite sequence; **Language** = set of strings ⊆ Σ\*.
- **FA** = 5-tuple (Q, Σ, δ, q₀, F); accepts if final state reached after reading the whole input.
- **DFA:** one next state. **NFA:** set of next states; accepts if *any* path accepts.
- **DFA ≡ NFA ≡ ε-NFA** in power (all give regular languages); NFA→DFA via **subset construction** (≤ 2ⁿ states).
- **Applications:** text search, lexical analysis, circuits, protocols.

## Frequently Asked Exam Questions
1. Define DFA and NFA. Differentiate between them.
2. Explain the central concepts: alphabet, string, language (with examples).
3. Design a DFA for strings over {0,1} that end with 01 / contain 00 / have even number of 1s.
4. Convert a given NFA to an equivalent DFA (subset construction).
5. Explain ε-NFA and ε-closure.
6. Write applications of finite automata.

---
*Sources: syllabus PDF (pp. 5–6); standard text Hopcroft–Motwani–Ullman Ch. 1–2; lecture notes on DFA/NFA/subset construction (Chalmers TMV027) and lexical analysis material.*
