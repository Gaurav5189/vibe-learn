# Artificial Intelligence (MCC-2.3.4) — Unit IV Notes
**PG MCA, Ravenshaw University (CBCS)** | Ref: Rich & Knight (Ch. 15.1–15.4, 5, 17); Patterson (Ch. 4.1–4.4, 5.3, 11.1, 11.4, 15); Russell & Norvig

**Unit IV syllabus:** Natural Language Processing: Introduction, Syntactic processing, Semantic analysis, Discourse and Pragmatic processing · Handling uncertainty: Probabilistic reasoning, Use of certainty factor · Knowledge organization & management: Introduction, HAM

---

# PART A — NATURAL LANGUAGE PROCESSING (NLP)

## 1. Introduction

**Natural Language Processing:** The field of AI concerned with enabling computers to **understand, interpret, and generate human (natural) languages** such as English or Hindi.

| Task | Meaning |
|---|---|
| **Natural Language Understanding (NLU)** | Map language input → internal meaning representation |
| **Natural Language Generation (NLG)** | Map internal representation → language output |

**Applications:** machine translation, question answering, chatbots/virtual assistants, speech recognition, text summarisation, sentiment analysis, information retrieval/extraction, spell and grammar checking, natural-language interfaces to databases.

**Why is NLP difficult? (ambiguity)**
| Type | Example |
|---|---|
| **Lexical** (word sense) | "bank" = river bank / financial bank |
| **Syntactic (structural)** | "I saw the man with the telescope" (who has the telescope?) |
| **Semantic** | "Time flies like an arrow" has multiple readings |
| **Referential** | "Ram told Shyam **he** won" (who is "he"?) |
| **Pragmatic** | "Can you pass the salt?" is a request, not a question about ability |

Other difficulties: idioms, metaphors, ellipsis, context dependence, errors/noise in speech and text, huge vocabulary, world knowledge needed.

### Levels (steps) of language processing (Rich & Knight)
```
 Input sentence
      │
 1. Morphological analysis   → words broken into root + affixes (printing → print + ing)
      │
 2. Syntactic analysis       → parse tree (grammatical structure)
      │
 3. Semantic analysis        → literal meaning (meaning representation)
      │
 4. Discourse integration    → meaning in light of preceding/following sentences
      │
 5. Pragmatic analysis       → intended meaning in context (what speaker really wants)
```
(Speech input adds a **phonological/acoustic** step first.) In practice, the stages interact and are not strictly sequential.

**Example from Rich & Knight:** "I want to print Bill's .init file."
- Morphological: "Bill's" → Bill + possessive; "printing" etc.
- Syntactic: identifies subject (I), verb (want), complement (to print Bill's .init file).
- Semantic: "Bill" → person named Bill; ".init file" → file owned by Bill.
- Discourse: which "Bill"? Which file? (use earlier context).
- Pragmatic: the sentence is a **command** to the system: print that file (not just a statement).

---

## 2. Syntactic Processing

**Syntactic analysis (parsing):** Converts a linear sequence of words into a **structure (parse tree)** showing how words relate to each other according to a **grammar**. It decides whether a sentence is grammatically valid.

### 2.1 Grammar: Context-Free Grammar (CFG)
A CFG has: **terminals** (words), **non-terminals** (S, NP, VP, …), **production rules**, and a **start symbol (S)**.

```
S   → NP VP
NP  → Det N | Det Adj N | Pronoun | NP PP
VP  → V NP | V | VP PP
PP  → Prep NP
Det → the | a        N → boy | apple | telescope
V   → ate | saw      Prep → with | in    Adj → red
```
**Parse tree for "The boy ate an apple":**
```
            S
        ┌───┴───┐
       NP       VP
     ┌─┴─┐    ┌─┴──┐
    Det  N    V    NP
    The boy  ate  ┌─┴─┐
                 Det  N
                 an  apple
```
Equivalent bracket form: [S [NP [Det the] [N boy]] [VP [V ate] [NP [Det an] [N apple]]]]

### 2.2 Parsing strategies
| | Top-Down parsing | Bottom-Up parsing |
|---|---|---|
| Starts from | Start symbol S | The words (input) |
| Direction | Expand rules downward to match words | Combine words upward into non-terminals to reach S |
| Driven by | Goals (grammar) | Data (input) |
| Pros | Never builds trees that can't lead to S | Never wastes effort on rules not matching input words |
| Cons | May explore many rules inconsistent with input; **left recursion** causes infinite loops | Builds many partial constituents that never fit into a sentence |

- **Backtracking:** when a choice fails, return and try another rule (depth-first).
- **Chart parsing (e.g., CYK, Earley):** stores partial results (dynamic programming) to avoid re-parsing the same sub-phrases; efficient and handles ambiguity.
- **Combination:** **Left-corner parsing** mixes both.

### 2.3 Transition networks
- **Finite-state transition network:** states connected by arcs labelled with word categories.
- **Recursive Transition Network (RTN):** arcs can be labelled with the name of another network (recursion), equivalent in power to CFG.
- **Augmented Transition Network (ATN)** (Woods, 1970; used in **LUNAR**): RTN + **registers** and **conditions/actions on arcs**; it can store features (number, tense), test agreement (subject-verb), and build the parse structure → handles context-sensitive aspects and builds deep structure.

### 2.4 Features and agreement
Extended grammars attach **features** (number, gender, person, tense) so that "The boys *runs*" is rejected (subject-verb agreement), e.g., unification-based grammars.

### 2.5 Output and problems
- **Output:** parse tree(s) passed to semantic analysis.
- **Problems:** structural **ambiguity** (several valid parses: PP attachment), ungrammatical or incomplete input, large grammars, efficiency.

---

## 3. Semantic Analysis

**Semantic analysis:** Assigns **meaning** to the syntactic structures: builds a **meaning representation** (independent of the surface wording) from the words and their structure.

**Syntax vs semantics:** "Colorless green ideas sleep furiously" is syntactically correct but semantically meaningless.

### 3.1 Tasks
- **Word sense disambiguation:** choose the right meaning of an ambiguous word using context.
- **Semantic roles:** determine who did what to whom, when, where, how.
- **Compositional semantics:** meaning of a phrase is built from the meanings of its parts and how they combine.
- **Selectional restrictions:** constraints that filter meanings (e.g., "eat" requires an animate agent and an edible object).

### 3.2 Approaches
| Approach | Idea |
|---|---|
| **Semantic grammars** | Grammar rules use **semantic categories** (e.g., `<Ship>`, `<Port>`) instead of purely syntactic ones; syntax and semantics processed together. Efficient but domain-specific (e.g., LIFER/SOPHIE). |
| **Case grammar (Fillmore)** | Each verb has **case roles** linking it to its arguments. |
| **Conceptual dependency (Schank)** | Map sentences to canonical primitive acts (ATRANS, PTRANS, MTRANS…) |
| **Logic-based (compositional)** | Translate into predicate logic / lambda calculus: "Every boy loves a girl" → ∀x boy(x) → ∃y girl(y) ∧ loves(x, y) |
| **Semantic networks / frames** | Represent meaning as nodes and links/slots |

### 3.3 Case grammar example
Sentence: **"John opened the door with a key."**
| Case role | Filler |
|---|---|
| Verb | open |
| **Agent** (doer) | John |
| **Object** (affected) | the door |
| **Instrument** | a key |
Other common cases: **Experiencer, Location, Source, Goal, Time, Beneficiary**.

### 3.4 Semantic ambiguity handling
Use selectional restrictions, domain knowledge, frequency/context, and feedback to the parser (a semantically incorrect parse is rejected).

---

## 4. Discourse Processing

**Discourse:** A coherent sequence of sentences (a text or dialogue). The meaning of a sentence often depends on the **preceding (and following) sentences**. **Discourse integration** links the meaning of the current sentence to that of the earlier ones.

### 4.1 Key issues
- **Coherence:** sentences relate logically (cause, result, elaboration, contrast, sequence).
- **Cohesion:** surface links (pronouns, connectives, repetition).
- **Reference resolution (anaphora):** deciding what pronouns and noun phrases refer to.

### 4.2 Types of reference
| Type | Example |
|---|---|
| **Anaphora** (refers back) | "Ram went to the market. **He** bought mangoes." |
| **Cataphora** (refers forward) | "When **he** arrived, Ram was tired." |
| **Definite noun phrase** | "I bought a car. **The car** is red." |
| **One-anaphora / ellipsis** | "I like the red one." / "John ate rice and Mary [ate] bread." |
| **Inferable (bridging)** | "I entered the house. **The door** was open." |

### 4.3 Resolving pronouns — heuristics/constraints
1. **Agreement:** gender, number, person must match.
2. **Recency:** prefer the most recently mentioned candidate.
3. **Syntactic role/parallelism:** subjects are preferred antecedents.
4. **Selectional/semantic constraints:** "The cat drank the milk. **It** was cold" → "it" = milk.
5. **World knowledge/inference** and **focus/centering** (the entity currently in focus).

### 4.4 Discourse structure and knowledge
- **Discourse structure:** segments and **coherence relations** (Hobbs; RST).
- **Focus/centering theory:** tracks the entity under discussion.
- **Scripts/frames** supply default events and participants (the restaurant script helps interpret "John ordered a steak").

---

## 5. Pragmatic Processing

**Pragmatics:** Study of language **in context**: the **intended meaning (speaker's goal)** beyond the literal meaning. Pragmatic analysis reinterprets the sentence using knowledge of the situation, the speaker, the hearer, and the world.

### 5.1 Speech acts (Austin, Searle)
Utterances are **actions**:
- **Locutionary act:** the literal utterance.
- **Illocutionary act:** the intended action (request, promise, warning…).
- **Perlocutionary act:** the effect on the hearer.

| Class | Purpose | Example |
|---|---|---|
| **Assertives** | State facts | "It is raining." |
| **Directives** | Get the hearer to act | "Close the door." |
| **Commissives** | Commit the speaker | "I will come tomorrow." |
| **Expressives** | Express feelings | "Thank you." |
| **Declarations** | Change reality | "I now pronounce you husband and wife." |

### 5.2 Indirect speech acts
"**Can you pass the salt?**" → literally a question about ability, pragmatically a **request**. "It's cold in here" → request to close the window.

### 5.3 Gricean maxims (cooperative principle)
Speakers are assumed to follow: **Quantity** (be as informative as required), **Quality** (be truthful), **Relation** (be relevant), **Manner** (be clear). Violations signal implied meaning (**implicature**).

### 5.4 Other pragmatic knowledge
- Context (time, place, participants), user's goals and beliefs (**user model**).
- **Plan recognition:** inferring the speaker's plan from the utterance.
- Politeness, tone, irony, sarcasm.

### 5.5 Comparison of the levels
| Level | Question answered | Output |
|---|---|---|
| Morphological | What are the word forms? | Root + affixes |
| Syntactic | Is it grammatical? What structure? | Parse tree |
| Semantic | What does it literally mean? | Meaning representation |
| Discourse | How does it relate to other sentences? | Resolved references, coherence |
| Pragmatic | What did the speaker intend? | Intended action/meaning |

---

# PART B — HANDLING UNCERTAINTY

## 6. Uncertainty in AI

**Uncertainty:** Real-world knowledge and data are often **incomplete, imprecise, noisy, inconsistent, or unreliable**, so conclusions cannot be drawn with certainty.

**Sources:** incomplete information (missing test results), unreliable sensors/observations, imprecise language ("tall"), inherent randomness, unreliable rules (heuristic rules "usually true"), conflicting experts.

**Why classical logic is not enough:** it handles only true/false and is **monotonic** (can't retract conclusions when new evidence arrives).

**Approaches:** **Probability theory** (Bayesian reasoning), **Certainty factors** (MYCIN), Fuzzy logic, Dempster–Shafer theory, non-monotonic reasoning.

---

## 7. Probabilistic Reasoning

### 7.1 Basics
- **Probability P(A):** degree of belief that event A occurs, **0 ≤ P(A) ≤ 1**; P(true) = 1, P(false) = 0.
- **Complement:** P(¬A) = 1 − P(A).
- **Addition rule:** P(A ∨ B) = P(A) + P(B) − P(A ∧ B).
- **Conditional probability:** **P(A | B) = P(A ∧ B) / P(B)** (probability of A given B).
- **Product rule:** P(A ∧ B) = P(A | B) · P(B).
- **Independence:** A, B independent if P(A ∧ B) = P(A)·P(B).
- **Joint probability distribution:** probabilities of all combinations of variable values; **marginalisation** sums out variables.

### 7.2 Bayes' theorem
**P(H | E) = P(E | H) · P(H) / P(E)**
- **P(H)**: prior probability of hypothesis H.
- **P(E | H)**: likelihood of evidence E if H is true.
- **P(H | E)**: **posterior** probability (belief after seeing E).
- **P(E)** = P(E|H)P(H) + P(E|¬H)P(¬H).

**Worked example (medical test):**
- 1% of people have a disease: P(D) = 0.01.
- Test is positive for 90% of diseased people: P(+|D) = 0.9.
- Test is positive for 5% of healthy people: P(+|¬D) = 0.05.
- P(+) = 0.9 × 0.01 + 0.05 × 0.99 = 0.009 + 0.0495 = 0.0585
- **P(D | +) = 0.009 / 0.0585 ≈ 0.154 (about 15%)**
- Even with a positive test, the probability is low because the disease is rare (prior matters).

**Multiple evidence (Naive Bayes):** assuming evidence is **conditionally independent** given the hypothesis:
P(H | E₁,…,Eₙ) ∝ P(H) · Π P(Eᵢ | H)

### 7.3 Bayesian (Belief) Networks
**Definition:** A **directed acyclic graph (DAG)** where:
- **Nodes** = random variables.
- **Arcs** = direct (causal) influence: A → B means A influences B.
- Each node has a **Conditional Probability Table (CPT)** P(Xᵢ | Parents(Xᵢ)).

**Joint distribution:** **P(X₁,…,Xₙ) = Π P(Xᵢ | Parents(Xᵢ))**

**Example (alarm network):**
```
 Burglary    Earthquake
      \        /
        Alarm
       /     \
  JohnCalls  MaryCalls
```
P(J, M, A, B, E) = P(J|A) · P(M|A) · P(A|B,E) · P(B) · P(E)

- Compact (stores far fewer numbers than the full joint table) and supports **diagnostic** (effect → cause), **causal** (cause → effect) and **intercausal** reasoning.
- **Inference:** enumeration, variable elimination, approximate (sampling) methods.

### 7.4 Advantages / Disadvantages
| Advantages | Disadvantages |
|---|---|
| Sound mathematical foundation | Needs many prior/conditional probabilities (hard to obtain) |
| Updates beliefs with new evidence (Bayes) | Independence assumptions (e.g., Naive Bayes) may be unrealistic |
| Bayesian networks reduce complexity | Computation is expensive for large networks |

---

## 8. Certainty Factors (CF)

**Certainty Factor:** A **numeric measure of belief** in a hypothesis given evidence, developed for **MYCIN** (Shortliffe & Buchanan) as a practical alternative to strict probability for rule-based systems. Used in expert systems where exact probabilities are unavailable.

### 8.1 Definition
- **CF[h, e] = MB[h, e] − MD[h, e]**
  - **MB** = Measure of Belief (increase in belief in h due to evidence e), 0–1.
  - **MD** = Measure of Disbelief, 0–1.
- **Range: −1 ≤ CF ≤ +1**
  - +1 = certainly true; −1 = certainly false; **0 = no evidence / unknown**.
- Rules carry a CF: `IF evidence THEN hypothesis (CF = 0.7)`.
- In MYCIN, a premise is considered **true only if its CF > 0.2** (threshold).

### 8.2 Combining certainty factors
**(a) Conjunction/Disjunction in the premise**
- **AND:** CF(A and B) = **min**(CF(A), CF(B))
- **OR:** CF(A or B) = **max**(CF(A), CF(B))
- **NOT:** CF(¬A) = −CF(A)

**(b) Rule application (chaining)**
**CF(conclusion) = CF(rule) × CF(premise)**
Example: rule CF = 0.8, premise CF = 0.5 → CF(conclusion) = **0.4**.

**(c) Combining two rules concluding the same hypothesis (CF₁, CF₂)**
| Case | Formula |
|---|---|
| Both positive | CF = CF₁ + CF₂ (1 − CF₁) |
| Both negative | CF = CF₁ + CF₂ (1 + CF₁) |
| Opposite signs | CF = (CF₁ + CF₂) / (1 − min(\|CF₁\|, \|CF₂\|)) |

**Examples:**
- CF₁ = 0.6, CF₂ = 0.5 (both positive) → 0.6 + 0.5(1 − 0.6) = 0.6 + 0.2 = **0.8**
- CF₁ = 0.6, CF₂ = −0.4 → (0.6 − 0.4) / (1 − 0.4) = 0.2 / 0.6 ≈ **0.33**

(Evidence accumulates: more supporting rules raise the CF toward, but never above, 1.)

### 8.3 Properties
- **Commutative and associative:** order of combining evidence does not matter.
- Values stay within [−1, +1].

### 8.4 Advantages / Disadvantages
| Advantages | Disadvantages |
|---|---|
| Simple, easy to compute | Ad hoc; not rigorously grounded in probability theory |
| Natural for experts to assign | Assumes independence of rules/evidence |
| No need for prior probabilities | Long reasoning chains degrade accuracy |
| Handles evidence for and against | Can give inconsistent results vs. true probabilities |
| Worked well in MYCIN (performance not very sensitive to exact CF values) | |

### 8.5 Probability vs Certainty Factor
| | Probability (Bayes) | Certainty factor |
|---|---|---|
| Range | 0 to 1 | −1 to +1 |
| Basis | Mathematical theory | Heuristic/empirical |
| Needs | Priors, likelihoods | Expert-assigned CF per rule |
| Disbelief | P(¬H) = 1 − P(H) (linked) | Belief and disbelief tracked separately |
| Used in | Bayesian networks, diagnosis | MYCIN and rule-based ESs |

---

# PART C — KNOWLEDGE ORGANIZATION AND MANAGEMENT

## 9. Introduction

**Knowledge organization:** how knowledge is **structured, stored, and indexed** inside a system so that relevant knowledge can be **retrieved quickly and used effectively**.
**Knowledge management:** the processes of **acquiring, representing, storing, retrieving, updating, validating, and maintaining** the knowledge base.

### 9.1 Why it matters
- Knowledge bases are **large**; searching all of them is too slow.
- Related knowledge must be **grouped** and **linked** (like human memory).
- Knowledge changes: it must be **updated** without contradictions.

### 9.2 Issues in knowledge organization
1. **Representation:** how to structure knowledge (rules, frames, networks, logic).
2. **Indexing and retrieval:** accessing relevant knowledge efficiently (hashing, keys, discrimination nets, associative links).
3. **Hierarchical organisation:** general-to-specific structuring with inheritance.
4. **Redundancy and consistency:** avoid duplicates, resolve contradictions (**truth maintenance**).
5. **Scalability:** performance as the KB grows.
6. **Knowledge acquisition bottleneck:** hard to extract expertise.

### 9.3 Organization methods
| Method | Idea |
|---|---|
| **Hierarchies / taxonomies** | Classes and subclasses (isa links) |
| **Associative networks** | Concepts linked by relations (semantic nets, HAM) |
| **Frames / schemas / scripts** | Grouped slots describing typical objects/situations |
| **Indexing** | Keys/hash tables/discrimination nets for fast access |
| **Clustering / chunking** | Group related items into larger units |
| **Modular/partitioned KBs** | Divide knowledge by domain/task |

### 9.4 Models of human memory (basis for AI memory organization)
- **Short-term (working) memory:** small capacity, temporary (~7 ± 2 items).
- **Long-term memory:** large, permanent.
  - **Episodic memory:** personal events and experiences.
  - **Semantic memory:** general facts and concepts.
  - **Declarative** ("knowing what") vs **procedural** ("knowing how") knowledge.
- Human memory is **associative**: one item triggers related items.

### 9.5 Knowledge management activities
| Activity | Description |
|---|---|
| **Acquisition** | Interviews, observation, protocol analysis, documents, machine learning |
| **Representation** | Encoding in a suitable formalism |
| **Validation/verification** | Checking correctness, completeness, consistency |
| **Storage & indexing** | Organising for efficient retrieval |
| **Retrieval & use** | Pattern matching, inference, search |
| **Update & maintenance** | Adding, modifying, deleting knowledge; version control |
| **Explanation/sharing** | Making knowledge accessible to users |

---

## 10. HAM (Human Associative Memory)

**Definition:** **HAM** is a **model of human long-term (semantic/episodic) memory** proposed by **John R. Anderson and Gordon H. Bower (1973)** in the book *Human Associative Memory*. It represents knowledge as a **network of associated concepts organised into propositions**, and explains how a person **encodes, stores, retrieves** and **infers** from sentence-level information. (Anderson later developed the **ACT** family of models from HAM.)

### 10.1 Key ideas
- Memory is an **associative network** of **nodes** (ideas/concepts) joined by **labelled links**.
- Information is stored as **propositions**; each proposition is a **binary tree** structure.
- A careful distinction is made between **words** (lexical items) and **concepts** (ideas); words point to concept nodes.
- Aims to represent the **meaning**, not the surface form, of sentences.

### 10.2 Structure of a proposition
A HAM proposition is built hierarchically:
```
 Proposition (fact)
 ├── Context (C)  ──┬── Location (L)
 │                  └── Time (T)
 └── Fact (F)     ──┬── Subject (S)
                    └── Predicate (P) ──┬── Relation (R)
                                        └── Object (O)
```
- **Context node:** where (L) and when (T) the fact holds.
- **Fact node:** the actual statement, split into **Subject (S)** and **Predicate (P)**.
- **Predicate** splits into **Relation (R)** (verb/relation) and **Object (O)**.
- Other link labels: **e** (element of a set), **∈** (instance/class membership).

**Example: "The hippie touched the debutante in the park."**
```
              Proposition
             /C          \F
        Context           Fact
        /L    \T         /S      \P
     park   past    hippie     Predicate
                              /R        \O
                           touch     debutante
```
Concepts such as *hippie*, *debutante*, *park* are **idea nodes**, linked to word nodes and to general concepts (hippie ∈ person, etc.).

### 10.3 Components / processes
| Component | Function |
|---|---|
| **Linguistic parser** | Converts an input sentence into a **tree structure** (input tree) |
| **Memory structure (long-term memory)** | The network of propositions and concepts |
| **MATCH process (encoding/retrieval)** | Matches the input tree against stored structure to find existing nodes; new information is **added** where it does not match |
| **Question answering / fact retrieval** | A question is parsed into a **probe tree** and matched against memory; matching node(s) give the answer |
| **Inference system** | Derives implicit facts (e.g., from "Spinoza is a man" and "all men have knees", answer "Does Spinoza have a knee?" = yes) |

**Working:** Sentence → parse into tree → match to memory → if found, retrieve/confirm; if not, **store** new associations.

### 10.4 Features / Evaluation
| Strengths | Limitations |
|---|---|
| Clear, structured representation of sentence meaning | Limited inference capability (early version) |
| Handles memory for **facts/sentences**; supports question answering | Mainly **declarative** knowledge; weak for procedural knowledge |
| Explains recognition, recall and some forgetting/interference | Rigid binary-tree propositions |
| Foundation for later cognitive models (**ACT, ACT-R**) | Early model; later superseded by richer architectures |

### 10.5 HAM vs semantic net
| | Semantic network | HAM |
|---|---|---|
| Unit | Nodes + labelled arcs | Propositions as binary trees (S, P, R, O, C, L, T) |
| Purpose | General knowledge representation | Model of **human** memory (psychological) |
| Context (time/place) | Not explicit | Explicit context part of each fact |
| Retrieval | Graph traversal | MATCH process with probe trees |

---

## 11. Important Exam Points
1. Steps in NLP (morphological, syntactic, semantic, discourse, pragmatic) with example.
2. Syntactic processing: CFG, parse tree, top-down vs bottom-up, ATN.
3. Semantic analysis: approaches (semantic grammar, case grammar), example.
4. Discourse processing: reference resolution / anaphora.
5. Pragmatics: speech acts, indirect speech acts.
6. Sources of uncertainty; Bayes' theorem with a numerical example; Bayesian network.
7. Certainty factors: definition, range, combining formulas, numerical example, MYCIN.
8. Probability vs certainty factor.
9. Knowledge organization and management: issues and methods.
10. HAM: structure of a proposition (with diagram), processes, strengths/limits.

---
*Sources: Rich & Knight "Artificial Intelligence" (NLP chapters: morphological → pragmatic steps); Woods (1970) on ATNs; Buchanan & Shortliffe "Rule-Based Expert Systems" (certainty factors); Anderson & Bower, "Human Associative Memory" (1973); Russell & Norvig "AIMA"; Patterson "Introduction to AI and Expert Systems".*
