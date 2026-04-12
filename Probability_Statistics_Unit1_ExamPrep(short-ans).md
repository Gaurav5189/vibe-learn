# Probability & Statistics — Unit 1: Exam Preparation Notes
### Format: 5 Questions × 1 Mark Each — Short Answer / Definition Style

---

> **How to use this file:**
> Each answer is written for **1 mark** — just the definition, formula, or a tiny example.
> In the exam, **2–3 lines** is all you need. Do NOT write long paragraphs.

---

## 1. Introduction to Probability

**Q: Define Probability.**

Probability is a numerical measure of the likelihood of occurrence of an event. It always lies between 0 and 1.

```
P(A) = Number of favourable outcomes / Total number of outcomes
     = n(A) / n(S)
```
where P(∅) = 0 (impossible) and P(S) = 1 (certain).

---

**Q: State Kolmogorov's Axioms of Probability.**

For any event A defined on sample space S:
```
1. P(A) ≥ 0
2. P(S) = 1
3. If A ∩ B = ∅, then P(A ∪ B) = P(A) + P(B)
```

---

## 2. Sample Space

**Q: Define Sample Space.**

The **sample space S** is the set of all possible outcomes of a random experiment.

> *Example:* Tossing a coin → S = {H, T}
> Rolling a die → S = {1, 2, 3, 4, 5, 6}

---

**Q: Define an Event.**

An **event** is any subset of the sample space S.

> *Example:* Getting an even number on a die → A = {2, 4, 6} ⊆ S.

---

**Q: What is a sure event and an impossible event?**

- **Sure event:** The entire sample space S; P(S) = 1.
- **Impossible event:** The empty set ∅; P(∅) = 0.

---

## 3. Probability of an Event

**Q: State the complement rule.**

```
P(Ā) = 1 − P(A)
```
> *Example:* P(rain) = 0.3  →  P(no rain) = 1 − 0.3 = 0.7

---

**Q: How is probability calculated using simple events?**

List all equally likely simple events in S, count the favourable ones:
```
P(A) = n(A) / n(S)
```
> *Example:* P(prime on a die) = 3/6 = 1/2,  since primes = {2, 3, 5}.

---

## 4. Counting Rules

**Q: State the Fundamental Counting Principle.**

If task 1 can be done in **m** ways and task 2 in **n** ways independently, both can be done in **m × n** ways.

> *Example:* 3 shirts × 4 pants = 12 different outfits.

---

**Q: Define Permutation and give its formula.**

A **permutation** is an arrangement of objects where **order matters**.
```
ⁿPᵣ = n! / (n − r)!
```
> *Example:* ⁵P₃ = 5!/2! = 60

---

**Q: Define Combination and give its formula.**

A **combination** is a selection of objects where **order does NOT matter**.
```
ⁿCᵣ = n! / [r! (n − r)!]
```
> *Example:* ⁵C₃ = 5!/(3!·2!) = 10

---

**Q: What is the difference between permutation and combination?**

| | Permutation | Combination |
|---|---|---|
| Order | Matters | Does not matter |
| Formula | n!/(n−r)! | n!/[r!(n−r)!] |
| Example | Arranging 3 books | Choosing 3 books |

---

## 5. Additive Rule

**Q: State the Additive Rule of Probability.**

For any two events A and B:
```
P(A ∪ B) = P(A) + P(B) − P(A ∩ B)
```
> *Example:* P(A)=0.6, P(B)=0.4, P(A∩B)=0.2 → P(A∪B) = 0.8

---

**Q: State the additive rule for mutually exclusive events.**

If A ∩ B = ∅ (mutually exclusive):
```
P(A ∪ B) = P(A) + P(B)
```

---

**Q: What are mutually exclusive events?**

Two events A and B are **mutually exclusive** if they cannot occur simultaneously.
```
A ∩ B = ∅  →  P(A ∩ B) = 0
```
> *Example:* Getting Head and Tail on a single coin toss.

---

## 6. Multiplication Rule

**Q: State the Multiplication Rule of Probability.**

For any two events A and B:
```
P(A ∩ B) = P(A) · P(B|A)
```
For **independent** events:
```
P(A ∩ B) = P(A) · P(B)
```

---

**Q: Define independent events.**

Events A and B are **independent** if the occurrence of one does not affect the probability of the other.
```
P(A|B) = P(A)   or equivalently   P(A ∩ B) = P(A) · P(B)
```

---

**Q: Distinguish between mutually exclusive and independent events.**

- **Mutually exclusive:** P(A ∩ B) = 0 — they cannot happen together.
- **Independent:** P(A ∩ B) = P(A)·P(B) — occurrence of one does not affect the other.

> Note: Mutually exclusive events (with P(A), P(B) > 0) are **never** independent.

---

## 7. Conditional Probability

**Q: Define Conditional Probability.**

The probability of event A **given** that event B has already occurred:
```
P(A|B) = P(A ∩ B) / P(B),    provided P(B) > 0
```

---

**Q: Give an example of conditional probability.**

> A die is rolled. Given the result is even, find P(result > 3).
```
B = {2,4,6},  A ∩ B = {4,6}
P(A|B) = (2/6) / (3/6) = 2/3
```

---

**Q: What is the multiplication rule derived from conditional probability?**

Rearranging P(A|B) = P(A∩B)/P(B):
```
P(A ∩ B) = P(B) · P(A|B)
```
This is the general multiplication rule.

---

## 8. Bayes' Rule

**Q: State Bayes' Theorem.**

If B₁, B₂, ..., Bₙ are mutually exclusive and exhaustive events, then:
```
P(Bₖ|A) = P(A|Bₖ) · P(Bₖ)  /  Σᵢ P(A|Bᵢ) · P(Bᵢ)
```

---

**Q: State the Law of Total Probability.**

If B₁, B₂, ..., Bₙ form a partition of the sample space:
```
P(A) = Σᵢ P(A|Bᵢ) · P(Bᵢ)
     = P(A|B₁)P(B₁) + P(A|B₂)P(B₂) + ... + P(A|Bₙ)P(Bₙ)
```

---

**Q: Define Prior and Posterior probability.**

- **Prior probability P(Bᵢ):** Probability of an event **before** observing any evidence.
- **Posterior probability P(Bᵢ|A):** Updated probability of an event **after** observing evidence A.

---

**Q: Give a one-line application of Bayes' theorem.**

> A defective item is found. Find probability it came from machine M₂:
```
P(M₂|D) = P(D|M₂)·P(M₂) / P(D)
```

---

## 9. Random Variable

**Q: Define a Random Variable.**

A **random variable (r.v.) X** is a function that assigns a **real number** to each outcome of a random experiment.
```
X : S → ℝ
```

---

**Q: What is a Discrete Random Variable?**

A r.v. X is **discrete** if it takes a **countable** number of values (finite or countably infinite).

> *Example:* Number of heads in 3 tosses → X ∈ {0, 1, 2, 3}

---

**Q: What is a Continuous Random Variable?**

A r.v. X is **continuous** if it can take **any value in an interval** (uncountably infinite values).

> *Example:* Height of a student, lifetime of a bulb → X ∈ [0, ∞)

---

**Q: Distinguish between discrete and continuous random variables.**

| | Discrete | Continuous |
|---|---|---|
| Values | Countable | Uncountable (interval) |
| P(X = x) | Can be > 0 | Always = 0 |
| Described by | PMF | PDF |

---

## 10. Probability Mass Function (PMF)

**Q: Define PMF.**

For a discrete r.v. X, the **PMF** p(x) is defined as:
```
p(x) = P(X = x)
```
Conditions:  p(x) ≥ 0  and  Σ p(x) = 1

---

**Q: What is a discrete probability distribution?**

A **discrete probability distribution** is a table or formula listing all possible values of a discrete r.v. X with their probabilities P(X = x), such that the probabilities sum to 1.

---

**Q: Give an example of a PMF.**

> Toss a fair coin twice. X = number of Heads.

| x | 0 | 1 | 2 |
|---|---|---|---|
| P(X=x) | 1/4 | 2/4 | 1/4 |

Sum = 1/4 + 2/4 + 1/4 = 1  ✓

---

## 11. Probability Density Function (PDF)

**Q: Define PDF.**

For a continuous r.v. X, f(x) is called the **PDF** if:
```
f(x) ≥ 0   and   ∫_{−∞}^{∞} f(x) dx = 1

P(a ≤ X ≤ b) = ∫_a^b f(x) dx
```

---

**Q: What is a continuous probability distribution?**

A **continuous probability distribution** is described by a PDF f(x), where probabilities are areas under the curve. For any single point, P(X = x) = 0; probability is only defined over intervals.

---

**Q: Give a small example of a PDF.**

> f(x) = 2x,  0 ≤ x ≤ 1  (0 otherwise)
```
∫_0^1 2x dx = [x²]_0^1 = 1  ✓  → Valid PDF

P(0 < X < 0.5) = ∫_0^{0.5} 2x dx = [x²]_0^{0.5} = 0.25
```

---

**Q: What is the key difference between PMF and PDF?**

- **PMF:** Used for discrete r.v. Gives actual probability at each point; P(X=x) > 0 possible.
- **PDF:** Used for continuous r.v. P(X = x) = 0 always; probability is only found over an interval using integration.

---

## 12. Cumulative Distribution Function (CDF)

**Q: Define CDF.**

The **CDF** of a random variable X is:
```
F(x) = P(X ≤ x),    −∞ < x < ∞
```
It gives the probability that X takes a value **less than or equal to** x.

---

**Q: State the properties of CDF.**

```
1. 0 ≤ F(x) ≤ 1
2. F(x) is non-decreasing
3. lim_{x→−∞} F(x) = 0
4. lim_{x→+∞} F(x) = 1
5. P(a < X ≤ b) = F(b) − F(a)
```

---

**Q: What is the relation between CDF and PDF?**

```
CDF from PDF:  F(x) = ∫_{−∞}^{x} f(t) dt

PDF from CDF:  f(x) = d/dx [F(x)]
```

---

**Q: What is the CDF of a discrete random variable?**

```
F(x) = Σ_{t ≤ x} p(t)
```
The CDF of a discrete r.v. is a **step (staircase) function** — it jumps at each value x where p(x) > 0.

---

**Q: How do you find P(a < X ≤ b) using the CDF?**

```
P(a < X ≤ b) = F(b) − F(a)
```
> *Example:* F(3)=0.8, F(1)=0.2 → P(1 < X ≤ 3) = 0.8 − 0.2 = 0.6

---

## 13. Joint Probability Distribution

**Q: Define Joint Probability Distribution.**

The **joint probability distribution** of two r.v. X and Y describes the probability of both taking specific values simultaneously.
```
Discrete:   p(x, y) = P(X = x, Y = y)
Continuous: f(x, y),  where ∫∫ f(x,y) dx dy = 1
```

---

**Q: What is a marginal probability distribution?**

The probability distribution of **one variable** obtained from the joint distribution by summing (discrete) or integrating (continuous) out the other variable.
```
Discrete:   p_X(x) = Σ_y p(x, y)
Continuous: f_X(x) = ∫_{−∞}^{∞} f(x, y) dy
```

---

**Q: When are X and Y independent in a joint distribution?**

X and Y are **independent** if their joint distribution equals the product of their marginals:
```
Discrete:    p(x, y) = p_X(x) · p_Y(y)
Continuous:  f(x, y) = f_X(x) · f_Y(y)
```

---

**Q: What conditions must a joint PMF satisfy?**

```
1. p(x, y) ≥ 0        for all (x, y)
2. Σ_x Σ_y p(x, y) = 1
```

---

## Quick 1-Mark Flashcard Revision

```
┌─────────────────────────────────────────────────────────────────┐
│  P(A) = n(A)/n(S)          P(Ā) = 1 − P(A)                     │
│  P(A∪B) = P(A)+P(B)−P(A∩B)                                     │
│  Mutually exclusive: P(A∩B) = 0                                 │
│  P(A∩B) = P(A)·P(B|A)  [Multiplication Rule]                   │
│  Independent: P(A∩B) = P(A)·P(B)                               │
│  P(A|B) = P(A∩B)/P(B)  [Conditional Probability]               │
│  Bayes: P(Bₖ|A) = P(A|Bₖ)P(Bₖ) / ΣP(A|Bᵢ)P(Bᵢ)               │
│  ⁿPᵣ = n!/(n−r)!    |    ⁿCᵣ = n!/[r!(n−r)!]                  │
│  PMF: Σp(x) = 1     |    PDF: ∫f(x)dx = 1                      │
│  CDF: F(x) = P(X≤x) |    f(x) = F'(x)                          │
│  P(a<X≤b) = F(b)−F(a)                                          │
│  Marginal: p_X(x) = Σ_y p(x,y)                                 │
│  Independence: p(x,y) = p_X(x)·p_Y(y)                          │
└─────────────────────────────────────────────────────────────────┘
```

---

*End of Unit 1 — Probability & Statistics (1-Mark Format)*
*Best of luck on your exam! 🎯*
