# Probability & Statistics — Unit 2: Exam Preparation Notes

---

## Table of Contents

1. [Bernoulli Random Variable](#1-bernoulli-random-variable)
2. [Mathematical Expectation](#2-mathematical-expectation)
3. [Independent Random Variables](#3-independent-random-variables)
4. [Covariance and Variance](#4-covariance-and-variance)
5. [Moment Generating Functions (MGF)](#5-moment-generating-functions-mgf)
6. [Markov's and Chebyshev's Inequality](#6-markovs-and-chebyshevs-inequality)
7. [Central Limit Theorem](#7-central-limit-theorem)
8. [★ Binomial Distribution (Exam Focus)](#8--binomial-distribution)
9. [★ Geometric Distribution (Exam Focus)](#9--geometric-distribution)
10. [★ Normal Distribution (Exam Focus)](#10--normal-distribution)
11. [★ Uniform Distribution (Exam Focus)](#11--uniform-distribution)
12. [★ Exponential Distribution (Exam Focus)](#12--exponential-distribution)
13. [Quick Reference Summary Table](#13-quick-reference-summary-table)

---

## 1. Bernoulli Random Variable

### Definition
A **Bernoulli random variable** models a single trial with exactly two outcomes: **Success (1)** with probability `p` and **Failure (0)** with probability `1 − p` (also written as `q`).

### PMF
```
P(X = x) = p^x · (1−p)^(1−x),    x ∈ {0, 1}
```
Or equivalently:
```
P(X = 1) = p,    P(X = 0) = 1 − p = q
```

### Mean, Second Moment, and Variance

**Mean (First Moment):**
```
E[X] = 0·(1−p) + 1·p = p
```

**Second Moment:**
```
E[X²] = 0²·(1−p) + 1²·p = p
```

**Variance:**
```
Var(X) = E[X²] − (E[X])² = p − p² = p(1−p) = pq
```

### Example
> A fair coin is tossed. Let X = 1 if Head, X = 0 if Tail. Then p = 0.5.
> E[X] = 0.5, Var(X) = 0.5 × 0.5 = 0.25.

---

## 2. Mathematical Expectation

### Discrete Case
If X is a discrete random variable with PMF P(X = xᵢ), then:
```
E[X] = Σ xᵢ · P(X = xᵢ)
```

For any function g(X):
```
E[g(X)] = Σ g(xᵢ) · P(X = xᵢ)
```

### Continuous Case
If X is a continuous random variable with PDF f(x), then:
```
E[X] = ∫ x · f(x) dx   (over the support of X)
```

For any function g(X):
```
E[g(X)] = ∫ g(x) · f(x) dx
```

### Properties of Expectation
| Property | Statement |
|---|---|
| Linearity | E[aX + b] = aE[X] + b |
| Sum | E[X + Y] = E[X] + E[Y] |
| Independence | If X, Y independent: E[XY] = E[X]·E[Y] |
| Constant | E[c] = c |

### n-th Moment
```
μ'ₙ = E[Xⁿ]   (n-th moment about origin)
μₙ = E[(X − μ)ⁿ]   (n-th central moment)
```

---

## 3. Independent Random Variables

Two random variables X and Y are **independent** if and only if:
```
P(X = x, Y = y) = P(X = x) · P(Y = y)   [Discrete]
f(x, y) = f_X(x) · f_Y(y)                [Continuous]
```

**Key consequence:**
```
E[XY] = E[X] · E[Y]   ⟹   Cov(X, Y) = 0
```
> **Note:** Cov(X,Y) = 0 does NOT imply independence, but independence DOES imply zero covariance.

---

## 4. Covariance and Variance

### Variance
```
Var(X) = E[(X − E[X])²] = E[X²] − (E[X])²
```

### Covariance
```
Cov(X, Y) = E[(X − μ_X)(Y − μ_Y)] = E[XY] − E[X]·E[Y]
```

### Variance of Sum
```
Var(X + Y) = Var(X) + Var(Y) + 2·Cov(X, Y)
```
If X, Y independent: `Var(X + Y) = Var(X) + Var(Y)`

### Correlation Coefficient
```
ρ(X, Y) = Cov(X, Y) / [σ_X · σ_Y],    −1 ≤ ρ ≤ 1
```

---

## 5. Moment Generating Functions (MGF)

### Definition
```
M_X(t) = E[e^(tX)]
```
- **Discrete:** M_X(t) = Σ e^(tx) · P(X = x)
- **Continuous:** M_X(t) = ∫ e^(tx) · f(x) dx

### Key Property — Moments from MGF
```
E[Xⁿ] = M_X^(n)(0) = [d^n/dt^n M_X(t)] evaluated at t=0
```

So:
- **Mean:** E[X] = M'_X(0)
- **Second Moment:** E[X²] = M''_X(0)
- **Variance:** Var(X) = M''_X(0) − [M'_X(0)]²

### Example
For Bernoulli(p):
```
M_X(t) = E[e^(tX)] = e^(t·0)·q + e^(t·1)·p = q + pe^t
M'_X(t) = pe^t  →  M'_X(0) = p  ✓ (Mean)
M''_X(t) = pe^t  →  M''_X(0) = p  ✓ (Second moment)
Var(X) = p − p² = pq  ✓
```

---

## 6. Markov's and Chebyshev's Inequality

### Markov's Inequality
For a **non-negative** random variable X and any a > 0:
```
P(X ≥ a) ≤ E[X] / a
```

**Proof:**
```
E[X] = ∫₀^∞ x f(x) dx ≥ ∫_a^∞ x f(x) dx ≥ a ∫_a^∞ f(x) dx = a · P(X ≥ a)
⟹ P(X ≥ a) ≤ E[X]/a
```

**Example:** If E[X] = 10, then P(X ≥ 50) ≤ 10/50 = 0.2.

---

### Chebyshev's Inequality
For any random variable X with mean μ and variance σ², and for any k > 0:
```
P(|X − μ| ≥ k) ≤ σ² / k²
```
Equivalently, with k = nσ:
```
P(|X − μ| ≥ nσ) ≤ 1/n²
```

**Proof (uses Markov's inequality):**
```
P(|X − μ| ≥ k) = P((X − μ)² ≥ k²)
Applying Markov's to Y = (X − μ)²:
≤ E[(X − μ)²] / k² = σ²/k²   □
```

**Example:** Var(X) = 4, μ = 10. Find upper bound for P(|X − 10| ≥ 6).
```
P(|X − 10| ≥ 6) ≤ 4/6² = 4/36 = 1/9 ≈ 0.111
```

---

## 7. Central Limit Theorem

### Statement
Let X₁, X₂, ..., Xₙ be i.i.d. random variables with mean μ and variance σ². Define the sample mean:
```
X̄ₙ = (X₁ + X₂ + ... + Xₙ) / n
```
Then as n → ∞:
```
Z = (X̄ₙ − μ) / (σ/√n)  →  N(0, 1)   in distribution
```

Or equivalently, the sum Sₙ = X₁ + ... + Xₙ satisfies:
```
(Sₙ − nμ) / (σ√n)  →  N(0, 1)
```

### Significance
- No assumption about the shape of the original distribution needed (only finite mean & variance).
- Foundation for most statistical inference (confidence intervals, hypothesis tests).
- Works well for n ≥ 30 in practice.

### Example
> Toss a fair coin 100 times. What is the approximate probability of getting between 45 and 55 heads?
> 
> X = number of heads ~ Binomial(n=100, p=0.5)
> μ = np = 50,  σ² = np(1−p) = 25,  σ = 5
> 
> P(45 ≤ X ≤ 55) ≈ P((45−50)/5 ≤ Z ≤ (55−50)/5)
>                 = P(−1 ≤ Z ≤ 1)
>                 ≈ 0.6827

---

---

# ★ EXAM-FOCUS DISTRIBUTIONS

> The following five distributions are the primary exam topics. Each includes: definition, PDF/PMF, derivation of mean/variance, MGF, and solved examples.

---

## 8. ★ Binomial Distribution

### Definition
A **Binomial distribution** arises when:
1. There are `n` independent trials.
2. Each trial results in success (p) or failure (q = 1−p).
3. X = number of successes in n trials.

**Notation:** X ~ B(n, p)

---

### PMF
```
P(X = x) = C(n, x) · pˣ · (1−p)^(n−x),    x = 0, 1, 2, ..., n
```
where `C(n,x) = n! / [x!(n−x)!]` is the binomial coefficient.

**Verification (Sum = 1):**
```
Σ_{x=0}^{n} C(n,x) pˣ qⁿ⁻ˣ = (p + q)ⁿ = 1ⁿ = 1   ✓
```

---

### Derivation of Mean (First Moment)

```
E[X] = Σ_{x=0}^{n} x · C(n,x) · pˣ · qⁿ⁻ˣ

Since x·C(n,x) = n·C(n−1, x−1):

E[X] = n·p · Σ_{x=1}^{n} C(n−1, x−1) · pˣ⁻¹ · qⁿ⁻ˣ

Let y = x − 1:

E[X] = n·p · Σ_{y=0}^{n−1} C(n−1, y) · pʸ · qⁿ⁻¹⁻ʸ
     = n·p · (p + q)ⁿ⁻¹
     = n·p · 1 = np

∴ E[X] = np
```

---

### Derivation of Variance

**Step 1: Find E[X(X−1)]**
```
E[X(X−1)] = Σ_{x=0}^{n} x(x−1) · C(n,x) · pˣ · qⁿ⁻ˣ

Using x(x−1)·C(n,x) = n(n−1)·C(n−2, x−2):

E[X(X−1)] = n(n−1)p² · Σ_{x=2}^{n} C(n−2, x−2) pˣ⁻² qⁿ⁻ˣ
           = n(n−1)p² · (p+q)ⁿ⁻²
           = n(n−1)p²
```

**Step 2: Find E[X²]**
```
E[X²] = E[X(X−1)] + E[X] = n(n−1)p² + np
```

**Step 3: Variance**
```
Var(X) = E[X²] − (E[X])²
       = n(n−1)p² + np − (np)²
       = n²p² − np² + np − n²p²
       = np − np²
       = np(1−p)

∴ Var(X) = npq
```

---

### Moment Generating Function (MGF)

**Derivation:**
```
M_X(t) = E[e^(tX)] = Σ_{x=0}^{n} e^(tx) · C(n,x) · pˣ · qⁿ⁻ˣ
        = Σ_{x=0}^{n} C(n,x) · (pe^t)ˣ · qⁿ⁻ˣ
        = (pe^t + q)ⁿ            [by Binomial Theorem]

∴ M_X(t) = (q + pe^t)ⁿ
```

**Mean from MGF:**
```
M'_X(t) = n(q + pe^t)ⁿ⁻¹ · pe^t
M'_X(0) = n(q + p)ⁿ⁻¹ · p = n·1·p = np   ✓
```

**Second Moment from MGF:**
```
M''_X(t) = npe^t · (n−1)(q+pe^t)ⁿ⁻² · pe^t + n(q+pe^t)ⁿ⁻¹ · pe^t
M''_X(0) = n(n−1)p² + np

Var(X) = M''_X(0) − [M'_X(0)]²
       = n(n−1)p² + np − n²p²
       = npq   ✓
```

---

### Solved Exam Examples

**Example 1 (Standard):**
> A biased coin has P(Head) = 0.4. It is tossed 10 times. Find:
> (a) P(X = 3)   (b) P(X ≤ 1)   (c) Mean and Variance

*Solution:*
```
n = 10, p = 0.4, q = 0.6

(a) P(X = 3) = C(10,3) · (0.4)³ · (0.6)⁷
             = 120 · 0.064 · 0.0279936
             ≈ 0.2150

(b) P(X ≤ 1) = P(X=0) + P(X=1)
             = (0.6)¹⁰ + 10·(0.4)·(0.6)⁹
             = 0.00605 + 0.04031
             ≈ 0.0464

(c) Mean = np = 10 × 0.4 = 4
    Variance = npq = 10 × 0.4 × 0.6 = 2.4
```

**Example 2 (Exam-style proof question):**
> Prove that the mean of a Binomial distribution B(n, p) is np.

*(Use the derivation shown above.)*

---

## 9. ★ Geometric Distribution

### Definition
The **Geometric distribution** models the number of trials needed to get the **first success** in a sequence of independent Bernoulli trials.

**Two common conventions:**
- X = number of trials until first success (inclusive): support {1, 2, 3, ...}
- X = number of failures before first success: support {0, 1, 2, ...}

> *We use the first convention (most common in Indian university exams): X = trial number of first success.*

**Notation:** X ~ Geo(p)

---

### PMF
```
P(X = x) = (1−p)^(x−1) · p,    x = 1, 2, 3, ...
```

**Interpretation:** The first x−1 trials are failures (each with probability q = 1−p), and trial x is a success (probability p).

**Verification:**
```
Σ_{x=1}^{∞} q^(x−1) · p = p · Σ_{x=1}^{∞} q^(x−1) = p · (1/(1−q)) = p · (1/p) = 1   ✓
```

---

### Memoryless Property
The Geometric distribution is the only discrete memoryless distribution:
```
P(X > m + n | X > m) = P(X > n)
```
**Proof:**
```
P(X > k) = Σ_{x=k+1}^{∞} q^(x−1)·p = p·q^k/(1−q) = q^k

P(X > m+n | X > m) = P(X > m+n) / P(X > m) = q^(m+n) / q^m = q^n = P(X > n)   □
```

---

### Derivation of Mean (First Moment)

```
E[X] = Σ_{x=1}^{∞} x · q^(x−1) · p = p · Σ_{x=1}^{∞} x · q^(x−1)

Using the identity: Σ_{x=1}^{∞} x · rˣ⁻¹ = 1/(1−r)²  for |r| < 1:

E[X] = p · 1/(1−q)² = p · 1/p² = 1/p

∴ E[X] = 1/p
```

---

### Derivation of Second Moment and Variance

**Second Moment:**
```
E[X²] = Σ_{x=1}^{∞} x² · q^(x−1) · p

Write x² = x(x−1) + x:

E[X²] = E[X(X−1)] + E[X]

E[X(X−1)] = p · Σ_{x=2}^{∞} x(x−1) · q^(x−1)
           = pq · Σ_{x=2}^{∞} x(x−1) · q^(x−2)
           = pq · 2/(1−q)³         [using d²/dq² of Σqˣ]
           = pq · 2/p³
           = 2q/p²

E[X²] = 2q/p² + 1/p
```

**Variance:**
```
Var(X) = E[X²] − (E[X])²
       = 2q/p² + 1/p − 1/p²
       = (2q + p − 1)/p²
       = (2q − q)/p²              [since p + q = 1, so p − 1 = −q]
       = q/p²

∴ Var(X) = (1−p)/p²
```

---

### Moment Generating Function (MGF)

**Derivation:**
```
M_X(t) = E[e^(tX)] = Σ_{x=1}^{∞} e^(tx) · q^(x−1) · p

       = pe^t · Σ_{x=1}^{∞} (qe^t)^(x−1)

       = pe^t · 1/(1 − qe^t)     [geometric series, valid for |qe^t| < 1]

∴ M_X(t) = pe^t / (1 − qe^t)
```

**Mean (μ'₁) from MGF:**
```
M'_X(t) = d/dt [ pe^t · (1 − qe^t)^(−1) ]

         = pe^t(1−qe^t)^(−1) + pe^t · qe^t · (1−qe^t)^(−2)

At t = 0:
M'_X(0) = p/p  +  pq/p²
         = 1  +  q/p
         = (p + q)/p  =  1/p

∴ μ'₁ = Mean = 1/p   ✓
```

**Second Moment (μ'₂) from MGF:**
```
Differentiating M'_X(t) once more and setting t = 0:

M''_X(0) = (2 − p)/p²

∴ μ'₂ = (2 − p)/p²
```

**Variance:**
```
Var(X) = μ'₂ − (μ'₁)²
       = (2 − p)/p²  −  1/p²
       = (1 − p)/p²  =  q/p²

∴ Var(X) = q/p²
```

---

### Solved Exam Examples

**Example 1:**
> A fair die is rolled repeatedly until a 6 appears. Find the mean and variance.

*Solution:*
```
p = 1/6,  q = 5/6

Mean    = 1/p  = 6 rolls
Var(X)  = q/p² = (5/6)·36 = 30
```

**Example 2:**
> A student passes with probability 0.4 each attempt. Find:
> (a) P(passes on exactly 3rd attempt)
> (b) P(passes within first 3 attempts)

*Solution:*
```
p = 0.4,  q = 0.6

(a) P(X = 3) = q²·p = (0.36)(0.4) = 0.144

(b) P(X ≤ 3) = 0.4 + 0.24 + 0.144 = 0.784
```

---

## 10. ★ Normal Distribution

### Definition
The **Normal (Gaussian) distribution** is the most important continuous distribution. A r.v. X follows Normal distribution if its PDF is bell-shaped and symmetric about the mean μ.

**Notation:** X ~ N(μ, σ²)

---

### PDF
```
f(x) = (1/σ√(2π)) · exp[−(x−μ)²/2σ²],    −∞ < x < ∞
```

**Standard Normal** (μ=0, σ²=1): Z ~ N(0,1):
```
φ(z) = (1/√(2π)) · e^(−z²/2)

Standardization:  Z = (X − μ)/σ
```

**Verification (integrates to 1):**
```
∫_{−∞}^{∞} f(x) dx

Let z = (x−μ)/σ → dx = σdz:

= (1/√(2π)) ∫_{−∞}^{∞} e^(−z²/2) dz

Using Gaussian integral = √(2π):

= (1/√(2π)) · √(2π) = 1   ✓
```

---

### Moment Generating Function (MGF)

**Step 1 — Standard Normal Z ~ N(0,1):**
```
M_Z(t) = (1/√(2π)) ∫_{−∞}^{∞} e^(tz) · e^(−z²/2) dz

Complete the square:  tz − z²/2 = t²/2 − (z−t)²/2

M_Z(t) = e^(t²/2) · (1/√(2π)) ∫_{−∞}^{∞} e^(−(z−t)²/2) dz

The integral = √(2π) (Normal PDF shifted, still integrates to 1):

∴ M_Z(t) = e^(t²/2)
```

**Step 2 — General X ~ N(μ, σ²):**  using X = μ + σZ:
```
M_X(t) = e^(tμ) · M_Z(σt) = e^(tμ) · e^(σ²t²/2)

∴ M_X(t) = exp(μt + σ²t²/2)
```

---

### Mean (μ'₁) from MGF
```
M'_X(t) = (μ + σ²t) · exp(μt + σ²t²/2)

At t = 0:
M'_X(0) = μ · e^0 = μ

∴ μ'₁ = Mean = μ   ✓
```

---

### Second Moment (μ'₂) from MGF
```
M''_X(t) = σ²·exp(μt + σ²t²/2)  +  (μ + σ²t)²·exp(μt + σ²t²/2)

At t = 0:
M''_X(0) = σ²·1  +  μ²·1  =  σ² + μ²

∴ μ'₂ = σ² + μ²
```

---

### Variance
```
Var(X) = μ'₂ − (μ'₁)²
        = (σ² + μ²) − μ²
        = σ²

∴ Var(X) = σ²   ✓
```

---

### Key Properties
```
1. Symmetric:  Mean = Median = Mode = μ
2. Total area under curve = 1
3. 68-95-99.7 Rule:
   P(μ − σ  < X < μ + σ)  ≈ 68.27%
   P(μ − 2σ < X < μ + 2σ) ≈ 95.45%
   P(μ − 3σ < X < μ + 3σ) ≈ 99.73%
4. Sum of independent normals is normal
```

---

### Solved Exam Examples

**Example 1:**
> X ~ N(50, 100). Find P(45 < X < 62).

*Solution:*
```
σ = √100 = 10

P(45 < X < 62) = P((45−50)/10 < Z < (62−50)/10)
               = P(−0.5 < Z < 1.2)
               = Φ(1.2) − Φ(−0.5)
               = 0.8849 − (1 − 0.6915)
               = 0.5764
```

**Example 2:**
> Heights are N(165, 100). Find P(height > 180 cm).

*Solution:*
```
P(X > 180) = P(Z > (180−165)/10)
           = P(Z > 1.5)
           = 1 − Φ(1.5)
           = 1 − 0.9332 = 0.0668  (6.68%)
```

---

## 11. ★ Uniform Distribution

### Definition
A r.v. X follows a **Uniform distribution** on [a, b] if every value in the interval is equally likely.

**Notation:** X ~ U(a, b)

---

### PDF and CDF
```
        ┌  1/(b−a),    a ≤ x ≤ b
f(x) =  │
        └  0,          otherwise


        ┌  0,              x < a
F(x) =  │  (x−a)/(b−a),   a ≤ x ≤ b
        └  1,              x > b
```

**Verification:**
```
∫_a^b 1/(b−a) dx = (b−a)/(b−a) = 1   ✓
```

---

### Moment Generating Function (MGF)
```
M_X(t) = ∫_a^b e^(tx) · (1/(b−a)) dx

       = (1/(b−a)) · [e^(tx)/t]_a^b

∴ M_X(t) = (e^(tb) − e^(ta)) / [t(b−a)],    t ≠ 0
```

---

### Mean (μ'₁) — Direct Integration
```
E[X] = ∫_a^b x · (1/(b−a)) dx

     = (1/(b−a)) · [x²/2]_a^b

     = (b² − a²) / (2(b−a))

     = (b−a)(b+a) / (2(b−a))

     = (a + b)/2

∴ μ'₁ = Mean = (a + b)/2
```

---

### Second Moment (μ'₂) — Direct Integration
```
E[X²] = ∫_a^b x² · (1/(b−a)) dx

      = (1/(b−a)) · [x³/3]_a^b

      = (b³ − a³) / (3(b−a))

      = (b−a)(b² + ab + a²) / (3(b−a))      [factoring b³−a³]

      = (a² + ab + b²)/3

∴ μ'₂ = (a² + ab + b²)/3
```

---

### Variance
```
Var(X) = μ'₂ − (μ'₁)²
        = (a² + ab + b²)/3  −  (a+b)²/4

LCM = 12:
        = [4(a² + ab + b²) − 3(a+b)²] / 12
        = [4a² + 4ab + 4b² − 3a² − 6ab − 3b²] / 12
        = (a² − 2ab + b²) / 12
        = (b − a)² / 12

∴ Var(X) = (b − a)²/12
```

---

### Solved Exam Examples

**Example 1:**
> X ~ U(2, 8). Find: (a) P(3 < X < 6)   (b) Mean   (c) Variance

*Solution:*
```
(a) P(3 < X < 6) = (6−3)/(8−2) = 3/6 = 1/2

(b) Mean = (2+8)/2 = 5

(c) Var(X) = (8−2)²/12 = 36/12 = 3
```

**Example 2:**
> A bus arrives every 15 min. A passenger arrives randomly. P(wait > 10 min)?

*Solution:*
```
X ~ U(0, 15)

P(X > 10) = (15−10)/15 = 5/15 = 1/3
```

---

## 12. ★ Exponential Distribution

### Definition
The **Exponential distribution** models the waiting time between events in a Poisson process (e.g., component lifetime, service time). It is the only continuous **memoryless** distribution.

**Notation:** X ~ Exp(λ),  λ > 0 is the rate parameter.

---

### PDF and CDF
```
        ┌  λe^(−λx),    x ≥ 0
f(x) =  │
        └  0,            x < 0

F(x) = 1 − e^(−λx),    x ≥ 0
```

**Verification:**
```
∫_0^∞ λe^(−λx) dx = λ · [−(1/λ)e^(−λx)]_0^∞ = λ · (1/λ) = 1   ✓
```

---

### Memoryless Property
```
P(X > s+t | X > s) = P(X > t)

Proof:
P(X > x) = e^(−λx)

P(X > s+t | X > s) = P(X > s+t) / P(X > s)
                   = e^(−λ(s+t)) / e^(−λs)
                   = e^(−λt)
                   = P(X > t)   □
```

---

### Moment Generating Function (MGF)

**Derivation:**
```
M_X(t) = ∫_0^∞ e^(tx) · λe^(−λx) dx

       = λ ∫_0^∞ e^(−(λ−t)x) dx          (valid for t < λ)

       = λ · [ e^(−(λ−t)x) / (−(λ−t)) ]_0^∞

       = λ · (0 − (−1/(λ−t)))

       = λ/(λ−t)

∴ M_X(t) = λ/(λ−t),    t < λ
```

---

### Mean (μ'₁) from MGF
```
M_X(t) = λ(λ−t)^(−1)

M'_X(t) = λ(λ−t)^(−2)         [chain rule: derivative of (λ−t)^(−1) w.r.t. t]

At t = 0:
M'_X(0) = λ/λ² = 1/λ

∴ μ'₁ = Mean = 1/λ   ✓
```

---

### Second Moment (μ'₂) from MGF
```
M''_X(t) = 2λ(λ−t)^(−3)

At t = 0:
M''_X(0) = 2λ/λ³ = 2/λ²

∴ μ'₂ = 2/λ²
```

---

### Variance
```
Var(X) = μ'₂ − (μ'₁)²
        = 2/λ²  −  1/λ²
        = 1/λ²

∴ Var(X) = 1/λ²
```

---

### Solved Exam Examples

**Example 1:**
> Lifetime X of a bulb ~ Exp(λ = 0.01). Find:
> (a) Mean lifetime   (b) P(X > 200)   (c) Variance

*Solution:*
```
(a) Mean = 1/0.01 = 100 hours

(b) P(X > 200) = e^(−0.01×200) = e^(−2) ≈ 0.1353

(c) Var(X) = 1/(0.01)² = 10,000 hours²
```

**Example 2 (Memoryless):**
> A machine has run 50 hours. What is P(it runs 30 more hours)?

*Solution:*
```
P(X > 80 | X > 50) = P(X > 30) = e^(−30λ)   [by memoryless property]
```

---

## 13. Quick Reference Summary Table

| Distribution | PMF / PDF | Mean (μ'₁) | Variance | MGF |
|---|---|---|---|---|
| **Bernoulli(p)** | pˣ(1−p)^(1−x) | p | pq | q+pe^t |
| **Binomial(n,p)** | C(n,x)pˣqⁿ⁻ˣ | np | npq | (q+pe^t)ⁿ |
| **Geometric(p)** | q^(x−1)·p | 1/p | q/p² | pe^t/(1−qe^t) |
| **Normal(μ,σ²)** | (1/σ√2π)e^[−(x−μ)²/2σ²] | μ | σ² | e^(μt+σ²t²/2) |
| **Uniform(a,b)** | 1/(b−a) | (a+b)/2 | (b−a)²/12 | (e^tb−e^ta)/t(b−a) |
| **Exponential(λ)** | λe^(−λx) | 1/λ | 1/λ² | λ/(λ−t) |

---

## Standard Formula Sheet (Exam Use)

```
══════════════════════════════════════════════════════
MOMENTS FROM MGF:
  μ'₁ = M'_X(0)          ← Mean
  μ'₂ = M''_X(0)         ← Second Moment
  Var = μ'₂ − (μ'₁)²    ← Variance

CHEBYSHEV:   P(|X−μ| ≥ k) ≤ σ²/k²
MARKOV:      P(X ≥ a) ≤ E[X]/a

STANDARDIZE: Z = (X−μ)/σ  ~  N(0,1)

MEMORYLESS:
  Geometric:   P(X>m+n|X>m) = P(X>n)
  Exponential: P(X>s+t|X>s) = P(X>t)
══════════════════════════════════════════════════════
```

---

*End of Unit 2 — Probability & Statistics Exam Notes*  
*Best of luck on your exam! 🎯*