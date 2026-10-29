# Machine Learning (MCC-2.3.2) — Unit IV: Neural Networks
**PG MCA, Ravenshaw University (CBCS) | Ref: S. Haykin, *Neural Networks: A Comprehensive Foundation*, 2e (Ch. 1, 3, 4, 9); Kohonen; Kosko (BAM); Fausett (associative memories)**

## Syllabus Topics
1. Models of a Neuron
2. Network Architectures
3. Least-Mean-Square (LMS) Algorithm
4. Perceptron and Perceptron Learning Algorithm
5. Multilayer Perceptron (MLP)
6. Back-Propagation Algorithm (BPA)
7. Heuristics for making BPA perform better
8. Accelerated convergence of BP learning
9. Conjugate Gradient Method
10. Quasi-Newton Methods
11. Self-Organizing Maps (SOM)
12. Associative Networks: Autoassociative Memory, Heteroassociative Memory, Bidirectional Associative Network (BAM)

---

## 1. Models of a Neuron

**Artificial Neural Network (ANN):** A massively parallel network of simple processing units (**neurons**) that learns from data by adjusting **synaptic weights**; knowledge is stored in the weights (Haykin's definition).

**Biological analogy:** dendrites (inputs) → cell body (summation) → axon (output); synapses = weights.

### 1.1 Basic Elements of an Artificial Neuron (Haykin's model)
1. **Synapses (weights)** `w_kj`: input `x_j` is multiplied by weight `w_kj` (positive = excitatory, negative = inhibitory).
2. **Adder (linear combiner):** `v_k = Σⱼ w_kj x_j + b_k`  (`b_k` = **bias**; acts as threshold, equivalent to an extra input `x₀ = +1` with weight `w_k0 = b_k`).
3. **Activation function** `φ(·)`: limits the amplitude of output: `y_k = φ(v_k)`.

```
x₁ ─w_k1─┐
x₂ ─w_k2─┤→ Σ (v_k = Σwx + b) → φ(v_k) → y_k
x_m─w_km─┘        ↑ bias b_k
```

### 1.2 Types of Activation Functions
| Function | Formula | Range | Notes |
|---|---|---|---|
| **Threshold (Heaviside / McCulloch–Pitts)** | `φ(v) = 1 if v ≥ 0, else 0` | {0,1} | Not differentiable; used in perceptron |
| **Signum** | `φ(v) = +1 if v>0, −1 otherwise` | {−1,+1} | Bipolar hard limiter |
| **Piecewise-linear** | linear in a range, saturates outside | [0,1] | Approximates amplifier |
| **Sigmoid (logistic)** | `φ(v) = 1/(1 + e^(−av))` | (0,1) | Differentiable; `φ' = aφ(1−φ)` |
| **Hyperbolic tangent** | `φ(v) = tanh(v)` (Haykin: `a·tanh(bv)`) | (−1,1) | Antisymmetric; `φ' = 1 − φ²`; preferred for BP |
| **ReLU** (modern) | `max(0, v)` | [0,∞) | Avoids vanishing gradient |
| **Linear** | `φ(v) = v` | ℝ | Output layer for regression |

**Stochastic neuron:** output is +1 with probability `P(v)` (sigmoid-shaped), else −1 (used in Boltzmann machines).

---

## 2. Network Architectures

| Architecture | Description | Example |
|---|---|---|
| **Single-layer feedforward** | Input layer connects directly to output layer of neurons; no hidden layer; signals flow one way | Perceptron, LMS (Adaline) |
| **Multilayer feedforward** | One or more **hidden layers** between input and output; fully/partially connected; extracts higher-order features | MLP |
| **Recurrent (feedback)** | At least one **feedback loop** (outputs fed back to inputs/other neurons, often via unit-delay `z⁻¹`); gives memory/dynamics | Hopfield net, BAM |
| **Lattice (competitive)** | Neurons arranged in a 1-D/2-D grid; each sees all inputs | SOM |

- **Notation:** an `m–h–n` network = m inputs, h hidden neurons, n outputs. Fully connected: every node of a layer connects to every node of the next.
- **Learning paradigms:** supervised (error-correction), unsupervised (Hebbian, competitive), reinforcement.
- **Learning rules (Haykin Ch. 2):** error-correction (delta), Hebbian, competitive, memory-based, Boltzmann.

---

## 3. Least-Mean-Square (LMS) Algorithm

Also called **Widrow–Hoff rule / delta rule**; trains a **single linear neuron (ADALINE)** in an **online** fashion.

**Setup:** output `y(n) = wᵀ(n) x(n)`; desired `d(n)`; error `e(n) = d(n) − y(n)`.
**Cost (instantaneous squared error):** `E(n) = ½ e²(n)`.

**Derivation (stochastic gradient descent):**
`∂E/∂w = −e(n) x(n)` ⇒

```
w(n+1) = w(n) + η · e(n) · x(n)
```
`η` = learning-rate parameter.

**Properties**
- Uses an **instantaneous estimate** of the gradient (noisy) instead of the true gradient (steepest descent / Wiener solution) — requires no knowledge of statistics, is simple, and is **model-independent & robust**.
- Does **not converge to a single point**; it converges in the mean to the **Wiener solution** `w₀ = R⁻¹p` and fluctuates around it (misadjustment).
- **Convergence in the mean:** `0 < η < 2/λ_max` (λ_max = largest eigenvalue of the input correlation matrix `R`). Small η → slow but accurate; large η → fast but noisy/unstable.
- Applications: adaptive filtering, noise/echo cancellation, channel equalisation, system identification.

---

## 4. Perceptron and Perceptron Learning Algorithm

### 4.1 Perceptron (Rosenblatt, 1958)
- The **simplest neural network for linearly separable pattern classification** — a single McCulloch–Pitts neuron (linear combiner + **hard limiter**).
- `v = Σ wᵢxᵢ + b` ; `y = sgn(v)` → classifies into two classes `C₁` (y=+1) and `C₂` (y=−1).
- **Decision boundary:** hyperplane `wᵀx + b = 0`.

### 4.2 Perceptron Learning Algorithm (PLA)
Absorb bias: `x = [+1, x₁..x_m]ᵀ`, `w = [b, w₁..w_m]ᵀ`; `d(n) = +1` for `C₁`, `−1` for `C₂`.
1. Initialise `w(0) = 0` (or small random values).
2. For each training sample `(x(n), d(n))`:
   - Compute `y(n) = sgn(wᵀ(n)x(n))`.
   - **Update:** `w(n+1) = w(n) + η [d(n) − y(n)] x(n)`
     (i.e., no change if correctly classified; add `ηx` if a `C₁` sample is misclassified; subtract `ηx` if a `C₂` sample is misclassified).
3. Repeat over the training set until no misclassifications.

**Perceptron Convergence Theorem:** If the two classes are **linearly separable**, PLA converges to a solution **in a finite number of iterations** (for any η > 0). If **not separable** (e.g., **XOR**), it never converges.

**Limitation:** cannot solve **XOR** (Minsky & Papert, 1969) → motivates **multilayer** networks.

| | LMS (Adaline) | Perceptron |
|---|---|---|
| Neuron | Linear | Hard-limiter |
| Error used | `d − v` (before threshold) | `d − y` (after threshold) |
| Convergence | Asymptotic, in the mean | Finite steps (if separable) |
| Cost | Minimises MSE | Corrects misclassifications |

---

## 5. Multilayer Perceptron (MLP)

**Definition:** A **feedforward network with one or more hidden layers** of (usually) **non-linear, differentiable** neurons (sigmoid/tanh), trained by **back-propagation**.

**Features (Haykin)**
1. Every neuron has a **smooth non-linear activation** (without it the MLP collapses to a single linear layer).
2. One or more **hidden layers** learn complex features progressively.
3. **High connectivity** (synaptic weights).

**Why MLP solves XOR:** hidden neurons transform inputs into a space where classes are linearly separable.

**Universal Approximation Theorem:** A single hidden layer with enough non-linear neurons can approximate any continuous function on a compact set to arbitrary accuracy.

**Two signal types:** **function signals** (forward, input → output) and **error signals** (backward, output → input).

---

## 6. Back-Propagation Algorithm (BPA)

**Idea:** Supervised learning for MLP using **gradient descent** on the error; the **chain rule** propagates the output error **backwards** to compute the gradient for every weight (the **generalised delta rule**).

**Notation:** neuron `j`, input from `i`: induced local field `v_j = Σᵢ w_ji y_i`, output `y_j = φ(v_j)`; error at output neuron `e_j = d_j − y_j`; instantaneous error energy `E(n) = ½ Σ_j e_j²(n)`; average `E_av = (1/N) Σ E(n)`.

**Weight correction (delta rule):**
```
Δw_ji(n) = η · δ_j(n) · y_i(n)
```
**Local gradient δ_j:**
- **Output neuron j:**  `δ_j = e_j · φ'(v_j)`
- **Hidden neuron j:**  `δ_j = φ'(v_j) · Σ_k δ_k w_kj`  (sum over neurons `k` in the next layer, using their δ's — this is the "back-propagation")

For sigmoid `φ(v) = 1/(1+e^(−v))`: `φ'(v) = y(1 − y)`. For `tanh`: `φ' = 1 − y²`.

**Algorithm**
1. **Initialise** weights with small random values.
2. **Forward pass:** present input, compute `v_j`, `y_j` layer by layer to the output; compute errors `e_j`.
3. **Backward pass:** compute `δ_j` for the output layer, then successively for each hidden layer back to the input.
4. **Update** weights: `w_ji ← w_ji + η δ_j y_i`.
5. Repeat for all patterns (an **epoch**) until the error is acceptably small / stopping criterion (e.g., change in `E_av` < threshold, or validation error rises — **early stopping**).

**Modes of training**
| | Sequential (online/stochastic) | Batch |
|---|---|---|
| Update | After **each** pattern | After the **whole epoch** (average gradient) |
| Storage | Less | More |
| Speed / escaping local minima | Faster, noisy (helps escape local minima), preferred for large redundant data | Accurate gradient, parallelisable |

**Generalised delta rule with momentum:** `Δw_ji(n) = α Δw_ji(n−1) + η δ_j(n) y_i(n)` (`0 ≤ α < 1`).

**Problems:** local minima, plateaus/flat regions, **vanishing gradients**, slow convergence, overfitting, choice of architecture.

---

## 7. Heuristics for Making the BPA Perform Better

(Practical tips — Haykin Ch. 4)
1. **Sequential vs batch:** sequential mode is usually faster for large, redundant datasets.
2. **Maximise information content:** present the examples with the **largest training error** or the **most different** from earlier ones; **randomise** the order each epoch.
3. **Activation function:** use an **antisymmetric** function (`φ(−v) = −φ(v)`), e.g., `φ(v) = a·tanh(bv)` with **a = 1.7159, b = 2/3** (so `φ(1) = 1`, `φ(−1) = −1`) → faster learning than the logistic function.
4. **Target values:** choose targets **inside the range** of the activation (offset ε from the asymptotes, e.g., ±(a − ε)) to avoid driving weights to infinity.
5. **Normalise inputs:** (a) remove the **mean**, (b) **decorrelate** (PCA), (c) **equalise covariance** (scale to similar variance).
6. **Initialisation:** small random weights drawn from a distribution with **zero mean** and variance `σ_w² = 1/m` (m = fan-in) so that `v_j` lies in the linear part of the sigmoid; avoid both too large (saturation) and too small (flat gradient) values.
7. **Learning from hints:** include prior knowledge about the function (invariances, symmetries).
8. **Learning rates:** different η per layer — **smaller for last layers**, larger for early layers; neurons with many inputs should have smaller η (`η ∝ 1/√fan-in`).
9. **Network pruning / regularisation:** weight decay, early stopping, **cross-validation**, optimal brain damage to avoid overfitting.
10. **Stopping criterion:** based on gradient norm, change in error, or validation set.

---

## 8. Accelerated Convergence of BP Learning

Standard BP (steepest descent) is slow. Speed-up techniques:

| Method | Idea |
|---|---|
| **Momentum** | Adds a fraction `α` of the previous update: smooths oscillations, accelerates along consistent gradient directions, helps cross flat regions |
| **Adaptive (variable) learning rate** | Increase η if error decreases, decrease η (and undo step) if error increases |
| **Delta-bar-delta (Jacobs' heuristics)** | Each weight has its **own η_ji(n)**; increase it linearly if the gradient sign stays the same across iterations; decrease it **exponentially** if the sign alternates |
| **Quickprop / RPROP (resilient BP)** | Uses only the **sign** of the gradient and a per-weight step size that grows/shrinks; ignores gradient magnitude |
| **Second-order methods** | Use curvature info: **Conjugate gradient, Quasi-Newton (BFGS), Levenberg–Marquardt** (see Sections 9 & 10) |
| Input normalisation, good initialisation | Reduce ill-conditioning of the error surface |

**Newton's method (basis of second-order methods):** `Δw = −H⁻¹ g`, where `g` = gradient, `H` = Hessian. Converges quickly (quadratically) near a minimum but needs the **Hessian computation + inversion — O(W³)** and storage O(W²) for W weights → impractical for large nets. CG and quasi-Newton approximate this cheaply.

---

## 9. Conjugate Gradient (CG) Method

**Idea:** Minimise the error function by moving along a sequence of **mutually conjugate (H-orthogonal) directions** `s(n)`, so progress made in earlier directions is **not undone**. For a quadratic function in `W` dimensions, CG reaches the minimum in **at most W steps**. It needs **only first-derivative (gradient) information** and **no Hessian storage** → suitable for large networks.

**Conjugate directions:** `sᵢᵀ H s_j = 0` for `i ≠ j`.

**Algorithm**
1. Start with the steepest-descent direction: `s(0) = r(0) = −g(0)`.
2. **Line search:** find step `η(n)` minimising `E(w(n) + η s(n))`; update `w(n+1) = w(n) + η(n) s(n)`.
3. New residual: `r(n+1) = −g(n+1)`.
4. New direction: `s(n+1) = r(n+1) + β(n+1) s(n)`
   - **Fletcher–Reeves:** `β = r(n+1)ᵀ r(n+1) / r(n)ᵀ r(n)`
   - **Polak–Ribière:** `β = r(n+1)ᵀ [r(n+1) − r(n)] / r(n)ᵀ r(n)` (generally better for non-quadratic problems)
5. **Restart** with steepest descent every `W` iterations (or when `β` is poor), because the error surface is not exactly quadratic.

**Compared with BP:** much faster convergence, no learning-rate or momentum to tune (uses line search), but a **batch method** with a more complex implementation; line search is costly. A variant, **Scaled Conjugate Gradient (SCG)**, avoids the line search.

---

## 10. Quasi-Newton Methods

**Idea:** Approximate the **inverse Hessian** `B ≈ H⁻¹` **iteratively using only gradient information**, so Newton-like convergence is obtained without computing or inverting the Hessian.

**Update step:** `w(n+1) = w(n) − η(n) B(n) g(n)` with `η(n)` from a line search, and `B(0) = I` (first step = steepest descent).

**Let** `s = w(n+1) − w(n)` (change in weights), `y = g(n+1) − g(n)` (change in gradient). They satisfy the **secant (quasi-Newton) condition:** `B(n+1) y = s`.

**BFGS update** (Broyden–Fletcher–Goldfarb–Shanno — the most popular):

```
B(n+1) = B + (1 + yᵀBy / sᵀy) · (s sᵀ)/(sᵀy) − (s yᵀB + B y sᵀ)/(sᵀy)
```
- Keeps `B` **symmetric positive definite** (so each step is a descent direction), if the line search is exact/adequate.
- **DFP** (Davidon–Fletcher–Powell) is an older, similar update; BFGS is more robust.
- **L-BFGS:** stores only the last few `(s, y)` pairs → memory-efficient for large problems.
- **Levenberg–Marquardt:** for sum-of-squares error, approximates Hessian by `JᵀJ + μI` (Gauss–Newton with damping); very fast for small/medium nets.

| Method | Info used | Storage | Speed | Best for |
|---|---|---|---|---|
| Steepest descent / BP | Gradient | O(W) | Slow | Large data, online |
| Conjugate gradient | Gradient (conjugate dirs.) | O(W) | Fast | Large networks |
| Quasi-Newton (BFGS) | Gradient (approx. inverse Hessian) | O(W²) | Faster | Medium networks |
| Newton | Exact Hessian | O(W²) + O(W³) | Fastest (per step) | Small problems |

---

## 11. Self-Organizing Maps (SOM) — Kohonen Network

**Definition:** An **unsupervised, competitive-learning** neural network that maps high-dimensional input patterns onto a **low-dimensional (usually 2-D) lattice** of neurons while **preserving topological relationships** — nearby inputs activate nearby neurons. Used for visualisation, clustering and feature discovery.

**Structure:** Input vector `x ∈ ℝᵐ` is connected to **every neuron** `j` of a lattice (each with weight vector `w_j ∈ ℝᵐ`). No hidden layer; no target outputs.

**Three processes (Haykin)**
1. **Competition:** the **winning neuron** (best-matching unit, BMU) for input `x` is the one with the smallest Euclidean distance:
   `i(x) = argmin_j ‖x − w_j‖`
2. **Cooperation:** the winner defines a **topological neighbourhood** in which neighbouring neurons are also excited, strongest at the winner and decaying with **lateral distance** `d_{j,i}`:
   `h_{j,i}(n) = exp( − d²_{j,i} / 2σ²(n) )`, with `σ(n) = σ₀ exp(−n/τ₁)` (neighbourhood **shrinks** over time).
3. **Synaptic adaptation:** weights of the winner and its neighbours move toward the input:

```
w_j(n+1) = w_j(n) + η(n) · h_{j,i(x)}(n) · [ x − w_j(n) ] ,   η(n) = η₀ exp(−n/τ₂)
```

**Algorithm**
1. Initialise weight vectors `w_j(0)` (small random values or sampled from inputs).
2. Draw an input `x` from the training set.
3. Find the BMU `i(x)` (competition).
4. Update weights of the BMU and neighbours (cooperation + adaptation).
5. Reduce `η(n)` and `σ(n)`; repeat from step 2 until the map stops changing noticeably (no. of iterations ≫ no. of neurons).

**Two phases of adaptation**
- **Ordering (self-organising) phase:** ~1000 iterations; large `η` (~0.1, decreasing), **large neighbourhood** (covering almost the whole lattice) → topological ordering forms.
- **Convergence phase:** many iterations (~500 × number of neurons); **small `η` (~0.01)** and **neighbourhood → just the winner** (or its nearest neighbours) → fine-tunes the map.

**Properties:** (1) *Approximation of the input space* by a set of prototypes (vector quantiser); (2) *Topological ordering*; (3) *Density matching* — more neurons in regions of higher input density; (4) *Feature selection* — captures the principal features of the data distribution.

**Visualisation:** **U-matrix** (distances between neighbouring neurons) shows cluster boundaries.

**SOM vs K-means:** K-means has no neighbourhood/topology; SOM with neighbourhood → 0 reduces to online K-means.

---

## 12. Associative Networks (Associative Memories)

**Associative memory:** stores **pattern associations** and **recalls** a stored pattern from a **partial or noisy cue** (content-addressable memory, unlike address-based computer memory). Training usually via **Hebb's rule / outer-product rule**: *"neurons that fire together wire together."*

**Types**
- **Autoassociative:** stored pairs `(s, s)`; recalls the **same** pattern (pattern completion, noise removal).
- **Heteroassociative:** stored pairs `(s, t)` with `s ≠ t`; maps input to a **different** output pattern.
- **Bidirectional (BAM):** heteroassociative **two-way** recall (`s → t` and `t → s`).

Use **bipolar** patterns (−1, +1). Convert binary to bipolar: `bipolar = 2·binary − 1`.

### 12.1 Autoassociative Memory
- **Training (Hebb / outer product):** for stored patterns `s(p)` (row vectors, `p = 1..P`):

```
W = Σ_p s(p)ᵀ s(p)         (n × n weight matrix; diagonal often set to 0)
```
- **Recall:** present (possibly noisy) `x`; compute `y_in = x W`; output `y = f(y_in)`, with `f(·) = +1 if ≥ 0, else −1` (bipolar threshold).
- Can be **iterative** (feed output back as input until stable) → **Hopfield network** is the recurrent, energy-based version of this idea.
- **Capacity:** roughly `P ≈ 0.15 n` for Hopfield-type nets; patterns should be mutually **orthogonal** for perfect recall. Beyond capacity → **crosstalk**, spurious states.

**Example:** store `s = [1 1 −1 −1]`: `W = sᵀs` (4×4), diagonal zeroed. Recall of `s` gives `s` back; a one-bit-flipped version is corrected.

### 12.2 Heteroassociative Memory
- Pairs `(s(p), t(p))` with `s ∈ ℝⁿ`, `t ∈ ℝᵐ`.
- **Training:** `W = Σ_p s(p)ᵀ t(p)`   (n × m)
- **Recall:** `y_in = x W`;  `y = f(y_in)` (threshold).
- Perfect recall if the stored input patterns are **orthogonal**; otherwise crosstalk errors.
- Applications: mapping an image to a label, a name to a face, translation tables.

### 12.3 Bidirectional Associative Memory (BAM) — Kosko, 1988
- A **two-layer, recurrent heteroassociative network** (layer X with n units, layer Y with m units) with **symmetric** connections: forward weights `W` (X→Y), backward weights `Wᵀ` (Y→X).
- **Training:** `W = Σ_p s(p)ᵀ t(p)` (bipolar) — same as heteroassociative.
- **Recall (two-way, iterative):**
  1. Present `x` at layer X (or `y` at layer Y).
  2. **Forward:** `y = f(x W)`.
  3. **Backward:** `x = f(y Wᵀ)`.
  4. Repeat 2–3 until `(x, y)` **stops changing** (stable/resonant pair).
- **Stability:** every BAM with threshold units converges to a **stable fixed point** (Kosko) — guaranteed by an **energy (Lyapunov) function** `E(x, y) = −x W yᵀ` that decreases (non-increases) at every update.
- **Capacity:** approximately `min(n, m)` stored pairs; limited by crosstalk; improved by **multiple training / dummy augmentation**.
- Variants: continuous BAM, adaptive BAM, multidirectional associative memory (MAM).

| | Autoassociative | Heteroassociative | BAM |
|---|---|---|---|
| Stored | `(s, s)` | `(s, t)` | `(s, t)` |
| Weight matrix | `Σ sᵀs` | `Σ sᵀt` | `Σ sᵀt` (+ `Wᵀ` backward) |
| Recall | One-way (or iterative) | One-way | **Two-way, iterative until stable** |
| Network type | Feedforward / recurrent | Feedforward | Recurrent (feedback) |
| Use | Noise removal, completion | Pattern → different pattern | Bidirectional association |

---

## Quick Revision Sheet

| Topic | One-line takeaway |
|---|---|
| Neuron | `y = φ(Σwx + b)`; activation: threshold, sigmoid, tanh |
| Architectures | Single-layer FF, multilayer FF, recurrent, lattice |
| LMS | `w ← w + η e x`; stochastic gradient; stable if `0<η<2/λmax` |
| Perceptron | Hard-limiter neuron; `w ← w + η(d−y)x`; converges iff linearly separable; fails XOR |
| MLP | Hidden layers + non-linear neurons; universal approximator |
| BPA | `Δw_ji = η δ_j y_i`; output `δ=eφ'`; hidden `δ=φ'Σδ_k w_kj` |
| Momentum | `Δw(n) = αΔw(n−1) + ηδy` |
| Heuristics | tanh `1.7159·tanh(2v/3)`, normalise inputs, small zero-mean init, randomise order |
| Accelerated BP | Momentum, adaptive η, delta-bar-delta, RPROP, second-order methods |
| Conjugate gradient | Conjugate directions, ≤ W steps for quadratic, β (FR/PR), line search |
| Quasi-Newton | BFGS approximates `H⁻¹` from `s=Δw`, `y=Δg` |
| SOM | Competition → cooperation → adaptation; topology-preserving; ordering & convergence phases |
| Autoassociative | `W = Σsᵀs`, recalls same pattern |
| Heteroassociative | `W = Σsᵀt`, different output |
| BAM | Two-way iterative recall through `W` and `Wᵀ`; energy `−xWyᵀ`; always stable |

## Likely Exam Questions
1. Explain the model of a neuron and the common activation functions.
2. Describe the different network architectures.
3. Derive the LMS algorithm. State its convergence condition.
4. Explain the perceptron learning algorithm and the convergence theorem. Why can't it solve XOR?
5. Derive the back-propagation algorithm (output and hidden local gradients).
6. List heuristics that improve BP performance.
7. Explain methods to accelerate BP convergence (momentum, delta-bar-delta, etc.).
8. Explain the conjugate gradient method and compare with steepest descent.
9. Explain quasi-Newton (BFGS) methods; why are they preferred over Newton's method?
10. Explain the SOM algorithm: competition, cooperation, adaptation.
11. Explain autoassociative, heteroassociative memory and BAM with training and recall.

---
*Aligned to the syllabus references: Haykin, Neural Networks (2e) — Ch. 1 (neuron model, architectures), Ch. 3 (LMS, perceptron, convergence theorem), Ch. 4 (MLP, back-propagation, heuristics, conjugate-gradient/quasi-Newton optimisation), Ch. 9 (SOM). Associative memories (auto/hetero/BAM) follow Kosko (1988) and standard texts such as Fausett.*
