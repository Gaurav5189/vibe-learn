# Artificial Intelligence (MCC-2.3.4) — Unit III Notes
**PG MCA, Ravenshaw University (CBCS)** | Ref: Rich & Knight (Ch. 17.1–17.6, expert systems material); Patterson (Ch. 11.1, 11.4, 15); Russell & Norvig

**Unit III syllabus:** Expert System: need & justification, Rule-based architecture, Non-production system architecture, Case studies: MYCIN, R1 · Learning: Concept of learning, Types of learning, Genetic algorithm, Neural network

---

# PART A — EXPERT SYSTEMS

## 1. Introduction

**Expert System (ES):** A computer program that embodies the **specialised knowledge of a human expert** in a narrow domain and uses it to solve problems, give advice, or make decisions at (near) expert level. Also called a **knowledge-based system**.

**Characteristics:**
- **Domain-specific** (narrow area), not general intelligence.
- **Separates knowledge from reasoning**: knowledge base + inference engine.
- Reasons with **symbolic knowledge** and **heuristics** (rules of thumb).
- Handles **uncertain/incomplete** information.
- Provides **explanations** of its reasoning (HOW / WHY).
- Can be updated by changing knowledge, not code.

**Expert System vs Conventional Program**
| | Conventional program | Expert system |
|---|---|---|
| Knowledge | Mixed with code | Separate knowledge base |
| Processing | Algorithmic, step by step | Heuristic, inference-based |
| Output | Always correct (if correct algorithm) | May be uncertain/probable |
| Data | Precise, complete | Can be incomplete/uncertain |
| Explanation | None | Can explain its reasoning |
| Modification | Hard (edit code) | Easy (add/change rules) |

**Expert System vs Human Expert**
| Human expert | Expert system |
|---|---|
| Perishable, retires/leaves | Permanent |
| Expensive, scarce | Cheaper, reproducible |
| Inconsistent (mood, fatigue) | Consistent |
| Creative, common sense, adapts | Not creative, brittle outside domain |
| Learns from experience naturally | Learning needs explicit updating |

---

## 2. Need and Justification for Expert Systems

**Why needed (need):**
1. **Scarcity of experts**: few specialists, in high demand.
2. **Knowledge preservation**: expertise is lost when experts retire or leave.
3. **Availability**: works 24×7, anywhere, can be copied to many sites.
4. **Consistency**: same decision for same inputs (no fatigue/emotion).
5. **Speed**: quick decisions in time-critical situations.
6. **Hazardous environments**: nuclear plants, space, deep sea.
7. **Complex domains**: too much knowledge for one person to retain/combine.
8. **Training**: used to train novices; explanation helps learning.

**Justification (when is an ES appropriate?):**
- The task requires **expert knowledge** but is **well-bounded** (narrow domain).
- Expertise is **available** (experts exist and can articulate their knowledge).
- Problem is solvable by **heuristic reasoning** (not purely numeric computation).
- **Payoff** is significant (cost savings, error reduction, time).
- Task is **not too easy** (conventional program suffices) nor **too hard** (needs common sense/creativity).
- Experts **disagree little** about solutions; task is **teachable** and has measurable success.

**Typical applications:**
| Category | Example |
|---|---|
| Diagnosis | MYCIN (medical), equipment fault finding |
| Configuration | R1/XCON (computer systems) |
| Interpretation | DENDRAL (chemical structure), PROSPECTOR (geology) |
| Planning/Scheduling | Production, project planning |
| Monitoring/Control | Plant/process control |
| Instruction | Tutoring systems |
| Financial | Loan approval, tax advice |

**Limitations:** knowledge acquisition bottleneck; brittle at domain edges; no common sense; difficult to maintain when very large; no real learning ability.

---

## 3. Rule-Based (Production System) Architecture

**Rule-based ES:** Knowledge is represented as **IF–THEN production rules**; reasoning is by a rule interpreter (inference engine).

### 3.1 Components
```
 Domain Expert ──> Knowledge Acquisition ──> ┌───────────────────┐
                    Module                   │  KNOWLEDGE BASE   │ (rules)
                                             └─────────┬─────────┘
 User <──> User Interface <──> INFERENCE ENGINE <──> WORKING MEMORY (facts)
                                  │
                                  └──> EXPLANATION FACILITY
```
| Component | Function |
|---|---|
| **Knowledge Base (KB)** | Permanent domain knowledge: IF–THEN rules (and facts). |
| **Working Memory (Global database)** | Facts about the *current case*: user inputs, intermediate conclusions. |
| **Inference Engine** | Reasoning mechanism: matches rules to facts, selects and fires rules. |
| **User Interface** | Dialogue with the user: questions, answers, advice. |
| **Explanation Facility** | Tells the user **WHY** a question is asked and **HOW** a conclusion was reached. |
| **Knowledge Acquisition Module** | Helps experts/knowledge engineers to add and edit knowledge. |

**People involved:** **Domain expert** (source of knowledge), **Knowledge engineer** (encodes it), **User** (consults the system).

### 3.2 Rule format
```
IF   condition1 AND condition2 ...   (antecedent / premise)
THEN conclusion / action             (consequent)
```
Example: `IF temperature > 38°C AND has_rash THEN suspect measles`

### 3.3 Inference engine cycle (recognise–act cycle)
1. **Match**: find all rules whose conditions are satisfied by working memory (conflict set).
2. **Conflict resolution**: select one rule (strategies below).
3. **Act (fire)**: execute the rule; update working memory.
4. Repeat until goal reached or no rule applies.

**Conflict resolution strategies:** rule order, **specificity** (most specific first), **recency** (newest data), **priority/salience**, **refractoriness** (don't fire the same rule on same data twice).

### 3.4 Reasoning strategies
| | Forward chaining | Backward chaining |
|---|---|---|
| Direction | Data-driven: facts → conclusions | Goal-driven: hypothesis → supporting facts |
| Starts with | Known facts | A goal/hypothesis |
| Best for | Monitoring, configuration, planning, interpretation | Diagnosis, classification |
| Example system | R1/XCON | MYCIN |

### 3.5 Advantages / Disadvantages
| Advantages | Disadvantages |
|---|---|
| Modular: rules added/removed independently | Many rules → slow matching, hard to maintain |
| Natural, readable knowledge | Rule interactions are hard to predict |
| Separation of knowledge and control | Poor for structured/hierarchical knowledge |
| Easy explanation (trace rules fired) | No learning; knowledge acquisition bottleneck |

**Expert system shell:** An ES with the domain knowledge **removed** (inference engine + interface + empty KB). Reuse by adding new rules. Example: **EMYCIN** (from MYCIN), CLIPS, OPS5.

---

## 4. Non-Production (Non-Rule-Based) System Architectures

When pure IF–THEN rules are inadequate, other architectures represent and use knowledge differently.

| Architecture | Idea | Features / Example |
|---|---|---|
| **Semantic network–based** | Knowledge as a graph of concept nodes and labelled links (isa, part-of). | Inheritance, easy association; e.g., **PROSPECTOR** used a semantic-network-style inference net (mineral exploration), **CASNET** |
| **Frame-based** | Knowledge in frames (slots, defaults, procedures) organised in a class hierarchy. | Structured, inheritance, default reasoning, procedural attachment; good for stereotyped objects/situations |
| **Logic-based (Prolog)** | Knowledge as logic clauses (Horn clauses); inference by resolution/unification. | Declarative, formal semantics; e.g., Prolog-based systems |
| **Blackboard architecture** | Independent specialist modules cooperate through a shared data area. | Multiple knowledge sources; e.g., **HEARSAY-II** (speech understanding) |
| **Case-based reasoning (CBR)** | Solve new problems by retrieving and adapting similar past cases. | Learning from experience; help desk, legal reasoning |
| **Object-oriented** | Knowledge as objects with attributes and methods; message passing. | Reuse, encapsulation |
| **Neural/hybrid** | Learned connections instead of explicit rules; combine with rules. | Pattern recognition; neuro-symbolic systems |

### Blackboard architecture (detailed)
Analogy: a group of specialists around a blackboard, each adding to the solution when they can contribute.
```
 KS1 ┐
 KS2 ├──> read/write ──> BLACKBOARD (shared global data, solution levels)
 KS3 ┘                          ▲
              CONTROL / SCHEDULER (decides which KS acts next)
```
- **Knowledge Sources (KS):** independent modules, each expert in one sub-area (own representation: rules, procedures...).
- **Blackboard:** shared workspace holding the evolving partial solutions, organised in **levels of abstraction**.
- **Control component (scheduler):** monitors the blackboard and chooses which KS runs next (opportunistic problem solving).
- **Advantages:** integrates diverse knowledge, handles uncertain/incomplete data, supports incremental solution building, parallelism.
- **Disadvantages:** complex control, difficult to test/debug.

**Production vs Non-production**
| | Rule-based | Non-production |
|---|---|---|
| Knowledge form | IF–THEN rules | Networks, frames, logic, cases, KS modules |
| Strength | Simplicity, modularity | Structure, richer relationships, cooperation |
| Weakness | Flat, unstructured knowledge | Complex design, less uniform |

---

## 5. Case Studies

### 5.1 MYCIN
**What:** One of the earliest and most influential expert systems; developed at **Stanford University** in the **early 1970s** by **Edward Shortliffe** (thesis) with **Bruce Buchanan** and others; written in **LISP**; evolved from DENDRAL.

**Domain:** Diagnosing **bacterial infections of the blood** (bacteremia, meningitis) and **recommending antibiotic therapy** with **dosage adjusted to the patient's body weight**. (Name from antibiotics ending in "-mycin".)

**Architecture (three subprograms):**
1. **Consultation program**: asks the physician questions about patient data (symptoms, lab results), reasons and recommends diagnosis + therapy.
2. **Explanation program**: answers **WHY** (why a question is asked) and **HOW** (how a conclusion was reached), by tracing rules.
3. **Knowledge acquisition program**: lets experts add/modify rules.

**Knowledge base:** about **500 production rules** (grew to roughly this number). Facts as *object–attribute–value* triples with a certainty factor.

**Sample rule (typical form):**
```
IF   the stain of the organism is gram-negative AND
     the morphology of the organism is rod AND
     the aerobicity of the organism is anaerobic
THEN there is suggestive evidence (CF 0.6) that the identity of the organism is bacteroides
```

**Control strategy:** **Goal-directed backward chaining**: start from the goal ("what is the organism / what therapy?"), find rules concluding it, ask the user for missing data.

**Uncertainty handling: Certainty Factors (CF)**
- CF ranges from **−1** (certainly false) to **+1** (certainly true); 0 = no evidence.
- Premise CF with AND = **min** of conditions' CFs; with OR = **max**.
- Conclusion CF = rule CF × premise CF.
- Two rules supporting the same conclusion (both positive): CF = CF₁ + CF₂(1 − CF₁).

**Performance:** In Stanford evaluations it performed at a level comparable to infectious-disease specialists and better than non-specialist physicians (commonly quoted accuracy ≈ 65%, vs ≈ 80% for experts).

**Contributions:** clear separation of KB and inference engine; **explanation facility**; uncertainty handling with CFs; spawned the shell **EMYCIN**; influenced later systems.
**Limitations:** never used in routine clinical practice (legal/ethical issues, computing needs, interface); small rule base; CF framework is ad hoc (not rigorous probability).

### 5.2 R1 / XCON (eXpert CONfigurer)
**What:** A **production-rule-based** expert system developed by **John McDermott** (Carnegie Mellon University) for **Digital Equipment Corporation (DEC)**, starting 1978; first used in DEC's plant in Salem, New Hampshire in 1980.

**Task:** Automatically **configure VAX computer systems**: from a customer's order, select compatible components and produce a complete layout (cabinets, boards, cables) with a specification of how to assemble them. Earlier conventional programs for the task had failed.

**Implementation:** Written in **OPS5** (rule-based language by Charles Forgy, using the efficient **Rete** matching algorithm).

**Architecture / control:**
- **Production memory:** configuration rules (state-transition operators).
- **Working memory:** the order and the partially built configuration.
- **Inference:** **forward chaining** (data-driven); rules fire when the current partial configuration matches; the system proceeds through a sequence of **subtasks** (check order, arrange components in cabinets/boxes, add backplanes/cables, etc.) until the configuration is complete.

**Scale and results:**
- Started with several hundred rules (~850 early), grew to about 2,500 rules by the mid-1980s, ~6,200 in 1987, and over 10,000 by 1989.
- By 1986 it had processed about 80,000 orders with ~95–98% accuracy; saved DEC tens of millions of dollars a year.
- Spawned related systems: **XSEL** (sales assistant), **XSITE** (site planning).

**Problem:** **Maintenance**: as rules grew, a large share changed each year; interactions became difficult to manage.

### MYCIN vs R1/XCON
| | MYCIN | R1/XCON |
|---|---|---|
| Developer | Stanford (Shortliffe, Buchanan) | CMU for DEC (McDermott) |
| Period | Early 1970s | 1978–80 onward |
| Domain | Medical diagnosis (blood infections) | Computer system configuration |
| Task type | Diagnosis/recommendation | Configuration/design |
| Language | LISP | OPS5 |
| Reasoning | Backward chaining | Forward chaining |
| Uncertainty | Certainty factors | Mostly certain (deterministic) |
| Rules | ~500 | 850 → 10,000+ |
| Explanation | Strong (WHY/HOW) | Limited |
| Deployment | Research prototype, not clinically used | Commercial, used in production by DEC |

---

# PART B — LEARNING

## 6. Concept of Learning

**Definitions:**
- **Simon (1983):** Learning denotes changes in a system that enable it to do the **same task (or tasks from the same population) more efficiently and effectively the next time**.
- **Mitchell (1997):** A program learns from **experience E** with respect to a class of **tasks T** and **performance measure P**, if its performance at T, as measured by P, improves with E.
- Example: Checkers: T = playing checkers; P = % games won; E = playing practice games.

**Why learning in AI?** Knowledge engineering is costly; environments change; systems can improve automatically and handle unforeseen situations.

**Components of a learning system:**
| Component | Role |
|---|---|
| **Performance element** | Performs the task using current knowledge |
| **Critic** | Gives feedback on how well it is doing (against a standard) |
| **Learning element** | Modifies the knowledge/performance element using feedback |
| **Problem generator** | Suggests new experiences to explore |

**Factors:** the learning environment/feedback, the representation of knowledge, the learning algorithm, and prior knowledge.

---

## 7. Types of Learning

### 7.1 By feedback / learning paradigm
| Type | Description | Examples |
|---|---|---|
| **Supervised learning** | Learns from **labelled** examples (input → correct output). | Classification, regression: decision trees, neural nets, SVM |
| **Unsupervised learning** | Finds **structure in unlabelled** data; no teacher. | Clustering (k-means), association, dimensionality reduction |
| **Reinforcement learning** | Agent learns by **rewards/penalties** from interaction with environment. | Game playing, robot control (Q-learning) |
| **Semi-supervised** | Few labelled + many unlabelled examples. | Web page classification |

### 7.2 By strategy (Rich & Knight)
| Type | Idea |
|---|---|
| **Rote learning** | Store computed results (memorisation) to avoid recomputation; e.g., Samuel's checkers program caching board values. |
| **Learning by taking advice** | Accept instructions from a teacher and convert them to usable form (operationalisation). |
| **Learning in problem solving** | Improve from own experience: **parameter adjustment**, **macro-operators** (store useful operator sequences), **chunking** (SOAR). |
| **Learning from examples (inductive learning/concept learning)** | Generalise from positive/negative examples to a concept: **version spaces**, **decision trees (ID3)**. |
| **Explanation-based learning (EBL)** | Use domain theory to explain why an example is an instance of a concept, then generalise from a single example. |
| **Learning by analogy** | Apply knowledge from a similar, known situation to a new problem. |
| **Discovery** | Explore and form new concepts/hypotheses without a teacher (e.g., AM, BACON). |
| **Neural network & genetic learning** | Connectionist and evolutionary learning (below). |

### 7.3 Inductive concept learning (brief)
- **Concept learning:** infer a general boolean function from training examples.
- **Version space / Candidate elimination:** maintains the **most specific (S)** and **most general (G)** hypotheses consistent with the examples; positive examples generalise S, negative examples specialise G.
- **Decision tree (ID3):** builds a tree choosing at each node the attribute with highest **information gain**.
  - Entropy: **H(S) = −Σ pᵢ log₂ pᵢ**
  - Gain(S, A) = H(S) − Σ (|S_v|/|S|)·H(S_v)

**Issues:** overfitting, noise, bias in hypothesis space.

---

## 8. Genetic Algorithm (GA)

**Definition:** A **search and optimisation technique** inspired by **natural selection and genetics** (John Holland, 1975). It evolves a **population** of candidate solutions through **selection, crossover, and mutation**, favouring fitter individuals ("survival of the fittest").

### 8.1 Terminology
| Term | Meaning |
|---|---|
| **Population** | Set of candidate solutions |
| **Chromosome (individual)** | One candidate solution, encoded as a string |
| **Gene** | One element/position of the chromosome |
| **Allele** | Value of a gene (e.g., 0 or 1) |
| **Encoding** | Representation of a solution (binary, integer, real, permutation) |
| **Fitness function** | Measures quality of a chromosome |
| **Generation** | One iteration of the algorithm |

### 8.2 Genetic operators
1. **Selection (reproduction):** choose parents with probability favouring higher fitness.
   - **Roulette wheel:** probability pᵢ = fᵢ / Σf.
   - **Tournament:** pick the best among k random individuals.
   - **Rank selection**, **elitism** (copy best individuals unchanged).
2. **Crossover (recombination):** combine two parents to make offspring (probability ~0.6–0.9).
   - **Single-point:** swap tails after one cut point.
   - **Two-point**, **uniform** crossover.
3. **Mutation:** randomly flip genes with small probability (~0.001–0.01); maintains diversity, avoids premature convergence.

### 8.3 Algorithm
```
1. Initialise a random population of N chromosomes
2. Evaluate fitness of each chromosome
3. Repeat until termination:
     a. Select parents (by fitness)
     b. Apply crossover to produce offspring
     c. Apply mutation to offspring
     d. Evaluate offspring; form new generation (replace population)
4. Return best chromosome found
```
**Termination:** max generations reached, fitness target met, or no improvement over several generations.

### 8.4 Worked example: maximise f(x) = x², x ∈ [0, 31] (5-bit strings)
| String | x | f(x) | Selection probability (f/Σf) |
|---|---|---|---|
| 01101 | 13 | 169 | 0.14 |
| 11000 | 24 | 576 | 0.49 |
| 01000 | 8 | 64 | 0.06 |
| 10011 | 19 | 361 | 0.31 |
| **Sum** | | **1170** | average = 292.5 |

- Fitter strings (11000, 10011) are selected more often.
- **Crossover** of 0110|1 and 1100|0 at point 4 → **01100** (x=12, f=144) and **11001** (x=25, f=625).
- **Mutation:** flip a random bit, e.g., 11001 → 11101 (x=29, f=841).
- Over generations the population moves toward **11111 (x = 31)**.

### 8.5 Features
| Advantages | Disadvantages |
|---|---|
| Works without gradient/derivative information | No guarantee of the global optimum |
| Handles large, complex, non-linear search spaces | Needs good encoding & fitness function |
| Parallel search through a population | Computationally expensive |
| Flexible; good for multi-objective problems | Parameter tuning (population size, rates) |

**Applications:** optimisation (scheduling, timetabling, routing/TSP), engineering design, feature selection, evolving neural networks, game strategies.

---

## 9. Neural Networks (Artificial Neural Networks, ANN)

**Definition:** A computational model inspired by the **brain**: a network of simple interconnected processing units (**neurons**) that learns from data by adjusting **connection weights**. Knowledge is stored in the weights (connectionist/sub-symbolic approach).

### 9.1 Biological vs artificial neuron
| Biological | Artificial |
|---|---|
| Dendrites | Inputs xᵢ |
| Synapse strength | Weights wᵢ |
| Cell body (soma) | Summation unit |
| Firing threshold | Activation function |
| Axon | Output y |

### 9.2 Artificial neuron (McCulloch–Pitts / perceptron unit)
```
 x1 ─w1─┐
 x2 ─w2─┼─> Σ (net = Σ wᵢxᵢ + b) ─> f(net) ─> y
 xn ─wn─┘            (b = bias)
```
- **net = w₁x₁ + w₂x₂ + … + wₙxₙ + b**, output **y = f(net)**.

**Activation functions:**
| Function | Formula | Range |
|---|---|---|
| Step (threshold) | y = 1 if net ≥ 0 else 0 | {0, 1} |
| Sigmoid (logistic) | y = 1 / (1 + e^(−net)) | (0, 1) |
| Tanh | (e^net − e^−net)/(e^net + e^−net) | (−1, 1) |
| ReLU | max(0, net) | [0, ∞) |

### 9.3 Network architectures
- **Single-layer feedforward:** inputs connect directly to output neurons (e.g., perceptron).
- **Multilayer feedforward (MLP):** input layer → one or more **hidden layers** → output layer; signals flow forward only.
- **Recurrent network:** feedback connections; has memory (e.g., Hopfield, RNN).

### 9.4 Perceptron (Rosenblatt, 1958)
Single neuron with step activation; a **linear classifier**.

**Learning rule:** **wᵢ ← wᵢ + η (t − y) xᵢ**, **b ← b + η (t − y)**
(η = learning rate, t = target, y = actual output).

- Converges if the classes are **linearly separable** (perceptron convergence theorem).
- Example: AND gate is linearly separable, so it is realised by w₁ = w₂ = 1, b = −1.5.
- **Limitation:** cannot solve **XOR** (not linearly separable) → needs a multilayer network (Minsky & Papert, 1969).

### 9.5 Learning in neural networks
| Paradigm | Method |
|---|---|
| **Supervised** | Perceptron rule, delta rule, **backpropagation** |
| **Unsupervised** | **Hebbian learning** ("neurons that fire together wire together": Δw = η·x·y), competitive learning, **Kohonen SOM** |
| **Reinforcement** | Weights adjusted by reward signals |

### 9.6 Backpropagation algorithm (for MLP)
Uses **gradient descent** to minimise error E = ½ Σ (t − o)².
1. **Initialise** weights with small random values.
2. **Forward pass:** compute outputs of hidden and output layers.
3. **Compute error** at the output: t − o.
4. **Backward pass:** propagate error back, computing local gradients (**deltas**):
   - Output neuron (sigmoid): **δₖ = (tₖ − oₖ) · oₖ(1 − oₖ)**
   - Hidden neuron: **δⱼ = oⱼ(1 − oⱼ) · Σₖ wⱼₖ δₖ**
5. **Update weights:** **Δwⱼᵢ = η · δⱼ · xᵢ** (optionally with momentum).
6. Repeat over all training examples (epochs) until error is acceptably small.

**Issues:** local minima, slow convergence, **overfitting** (use validation set, regularisation), choice of learning rate, number of hidden layers/neurons.

### 9.7 Advantages / Disadvantages / Applications
| Advantages | Disadvantages |
|---|---|
| Learn from examples; no explicit rules | "Black box": poor explainability |
| Handle noisy, incomplete data; generalise | Needs large training data and computing |
| Parallel processing; fault tolerant | Training may be slow/stuck in local minima |
| Model non-linear relationships | Choice of architecture/parameters is empirical |

**Applications:** pattern/character/speech recognition, image classification, forecasting, medical diagnosis, control systems, natural language processing.

---

## 10. Comparison: Genetic Algorithm vs Neural Network
| | Genetic Algorithm | Neural Network |
|---|---|---|
| Inspiration | Natural evolution | Brain neurons |
| Main role | Search / optimisation | Learning / pattern recognition |
| Representation | Population of encoded solutions | Weights of interconnected neurons |
| Operators | Selection, crossover, mutation | Weight updates (gradient descent, Hebb) |
| Learning type | Evolutionary, population-based | Supervised / unsupervised / reinforcement |
| Gradient needed | No | Usually yes (backprop) |

---

## 11. Important Exam Points
1. Define expert system; need and justification; ES vs conventional program.
2. Rule-based ES architecture with diagram; role of each component; forward vs backward chaining.
3. Non-production architectures: semantic net, frame, blackboard (with diagram).
4. MYCIN case study: domain, architecture, CF, backward chaining, results.
5. R1/XCON case study: domain, OPS5, forward chaining, results; MYCIN vs R1.
6. Concept of learning; components of a learning system.
7. Types of learning (supervised, unsupervised, reinforcement; rote, advice, examples, EBL, analogy).
8. Genetic algorithm: terminology, operators, algorithm, numeric example.
9. Neural network: neuron model, activation functions, perceptron rule, XOR limitation, backpropagation steps.

---
*Sources: Buchanan & Shortliffe, "Rule-Based Expert Systems" (MYCIN, Ch. 4, 15); McDermott, "R1: A Rule-Based Configurer of Computer Systems" (AI Journal, 1982) and related DEC/XCON references; Rich & Knight "Artificial Intelligence"; Patterson "Introduction to AI and Expert Systems"; Russell & Norvig "AIMA"; Holland (GA), Rosenblatt/Rumelhart (neural networks).*
