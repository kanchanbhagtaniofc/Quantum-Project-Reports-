# Quantum Advantage in Practice: The Quantum Hermite Transform and Fast-Forwarding the Quantum Harmonic Oscillator

*A technical report explaining, from first principles, and independently verifying via a from-scratch Qiskit implementation, the results of:*

*Jain, S., Iyer, V., Somma, R. D., Bao, N., Jordan, S. "Efficient Quantum Hermite Transform." Proceedings of the 58th Annual ACM Symposium on Theory of Computing (STOC '26), Salt Lake City, UT, June 22–26, 2026. https://doi.org/10.1145/3798129.3800772*

*as covered in Brookhaven National Laboratory's press release, ["Building out the Quantum Computing Toolkit"](https://www.bnl.gov/newsroom/news.php?a=223007) (July 13, 2026).*

---

## Executive Summary

In July 2026, Brookhaven National Laboratory publicized a new result from a Brookhaven/Northeastern University/Google Quantum AI/UT Austin collaboration: the **quantum Hermite transform (QHT)**, described as a new fundamental "primitive" for quantum algorithms — something the field has very few of, alongside the quantum Fourier transform (QFT) and amplitude amplification. This report explains what that means, why it matters, and — going beyond simply summarizing the paper — independently re-derives and verifies the result's central technical claim with a working Qiskit implementation, catching and fixing a real bug along the way.

The headline result is not a new attack on an existing system (as in most "quantum advantage" stories in cryptography); it is a new *capability*: an efficient way to change basis into the Hermite functions — the natural eigenbasis of the quantum harmonic oscillator (QHO), and a basis that shows up constantly in physics, signal processing, statistics, and Gaussian-distribution machine learning. The paper shows this basis change can be done with a circuit whose size is only **polylogarithmic** in the dimension of the space involved, matching the efficiency class of the QFT itself. The key technical enabler is a separate result of independent interest: the QHO can be **"fast-forwarded"** — its time evolution for *any* duration can be applied with a circuit of *constant* size, a property that, per general "no-fast-forwarding" theorems, essentially no Hamiltonian is guaranteed to have.

To ground this in something checkable rather than just reported, this document's companion repository, `qiskit-quantum-hermite-transform`, implements the paper's central fast-forwarding factorization directly in Qiskit and verifies it numerically: a **7-gate, depth-7 circuit** reproduces the exact time evolution of a discretized quantum harmonic oscillator with fidelity **≥ 0.999999** for evolution times ranging up to **200** (roughly 32 full oscillation periods), while a naive first-order Trotter simulation would need **almost 12,000** sequential steps to match that accuracy at the largest tested time. The companion project also verifies, to machine precision, that the discretized Hermite states the paper works with are genuine eigenstates of the discretized QHO Hamiltonian, and demonstrates the paper's "Hermite sampling" primitive end-to-end as an actual measured quantum circuit.

---

## 1. Why This Is a "Quantum Advantage" Story, and What Kind

Readers of my earlier survey, `Quantum_Advantage_Crypto_Survey.md`, will recall the taxonomy used there to classify quantum advantage claims:

- **Exponential advantage** — a problem moves from super-polynomial classical time to polynomial quantum time (Shor's algorithm is the canonical example).
- **Polynomial/sub-exponential advantage in the exponent** — the problem stays exponential, but the achievable exponent shrinks (Grover's algorithm; lattice sieving).
- **No known advantage** — the basis of post-quantum cryptography.

The Hermite transform result is a different flavor entirely, and worth naming explicitly: it is a **new primitive claiming provable *query* advantage for specific downstream tasks** (property testing and learning over the Gaussian distribution), built on top of a **classically-impossible-in-general capability** (Hamiltonian fast-forwarding) that happens to be achievable for this one, very important, physical system. It is not an attack. It does not threaten any deployed cryptographic system. It is closer in spirit to the QFT itself: a reusable *building block* whose downstream applications are still being discovered, several of which the paper's authors list explicitly as open problems (Section 6 below).

This is also, refreshingly, a case where "quantum advantage" is not resting entirely on unproven complexity-theoretic conjectures the way RSA/lattice/code-based hardness assumptions do. The core fast-forwarding claim (Theorem 1) is an unconditional, provable statement about circuit complexity for a specific, well-defined discretized Hamiltonian — the kind of claim that can be (and, in this report, is) checked directly.

---

## 2. Prerequisites: What You Need to Know to Understand This Result

### 2.1 The quantum harmonic oscillator (QHO)

The quantum harmonic oscillator is the quantum-mechanical version of a mass on a spring: a particle in a quadratic ("bowl-shaped") potential well. Its Hamiltonian (energy operator) in the continuum is

```
H = (x^2 + p^2) / 2
```

where `x` is the position operator and `p` is the momentum operator. It is one of the handful of quantum systems with an exact, closed-form analytical solution, and it appears — exactly or as an approximation — throughout physics: vibrating molecules, phonons in solids, modes of the electromagnetic field (relevant to quantum optics and the Jaynes–Cummings model discussed below), and as the base case for perturbation theory around almost any stable equilibrium.

Crucially for this paper, the QHO's energy eigenstates are known exactly: they are the **Hermite functions**, with eigenvalues `n + 1/2` for integer `n ≥ 0` (in units where `ℏ = ω = 1`).

### 2.2 Hermite polynomials and Hermite functions

The (physicist-normalized) Hermite functions are defined as

```
ψ_n(x) := (-1)^n / sqrt(2^n n! sqrt(π)) · e^{-x²/2} · H_n(x)
```

where `H_n(x)` is the `n`-th Hermite polynomial. These form a complete **orthonormal basis** for functions on the real line, weighted by a Gaussian envelope — which is exactly why they show up constantly in any setting involving Gaussian distributions: signal processing, statistics, and Gaussian-input machine learning theory, in addition to their role as the QHO's eigenbasis.

The **(classical) Hermite transform** is the change of basis from the standard "position" representation of a function into its representation as a sum of Hermite functions — directly analogous to how the (classical) Fourier transform re-expresses a function as a sum of sinusoids. Just as the Fourier transform is indispensable for anything involving periodic structure, the Hermite transform is the natural tool for anything with Gaussian structure.

### 2.3 The quantum Fourier transform, for comparison

The QFT is the quantum circuit implementing the (discrete) Fourier transform as a unitary operation on `n` qubits, in `O(n²)` gates (or `O(n log n)` with known parallelization tricks) — exponentially faster than any classical FFT applied to the same `2^n`-dimensional vector, because the quantum circuit acts on all `2^n` amplitudes *simultaneously* in superposition rather than one at a time. The QFT alone is not useful for extracting information (measuring after a QFT just gives you a random Fourier coefficient), but it is the engine inside Shor's algorithm, quantum phase estimation, and dozens of other quantum algorithms.

The paper's framing is direct: *if the QFT is the quantization of the classical Fourier transform, why not quantize the classical Hermite transform too?* — and the answer, until this paper, was that nobody had found an efficient way to do it.

### 2.4 Hamiltonian simulation and "fast-forwarding"

**Hamiltonian simulation** is the problem of implementing the time-evolution operator `e^{-iHt}` for a given Hamiltonian `H` and duration `t`, as a quantum circuit. This is the original 1982 Feynman motivation for quantum computers in the first place. The generic complexity of simulating a bounded Hamiltonian `H` for time `t` scales like `Ω(‖H‖t)` — the circuit gets bigger the longer you want to evolve the system, which is intuitively unavoidable in general (evolving twice as long should cost roughly twice as much).

**Fast-forwarding** means beating that generic lower bound: finding a circuit for `e^{-iHt}` whose size grows *sub-linearly* in `‖H‖t` — ideally not growing with `t` at all. This sounds like it should be broadly impossible, and mostly it is: the paper explains that if you could fast-forward an *arbitrary* Hamiltonian, you could simulate an arbitrary `t`-gate quantum circuit using fewer than `t` gates, which would collapse complexity classes (implying, e.g., BQP = PSPACE) — a strong signal that this is not generically achievable. Known "no-fast-forwarding" theorems make this rigorous for the worst case.

But *some* specific Hamiltonians, with enough special algebraic structure, genuinely can be fast-forwarded, and the QHO turns out to be one of them — which is the whole basis of this paper's construction.

### 2.5 Quantum phase estimation (QPE) and amplitude amplification

Two further standard primitives the paper's algorithm depends on:

- **Quantum phase estimation** extracts the eigenvalue (phase) associated with an eigenstate of a unitary, given the ability to apply controlled powers of that unitary. It's the workhorse behind Shor's algorithm's order-finding step and is used here (in a "fast" form enabled by the fast-forwarding result) to *verify* that a prepared state is close to the correct Hermite/QHO eigenstate.
- **Amplitude amplification** (the generalization of Grover's algorithm covered in my earlier crypto survey) is used here in its "fixed-point" variant (Yoder–Low–Chuang, 2014) to boost an approximately-correct state preparation into a highly accurate one, without the risk of "overshooting" that plain amplitude amplification has.

---

## 3. The Paper's Core Results

### 3.1 Problem statement

Working in a **discretized, finite-dimensional** version of the problem (since a real quantum circuit needs a finite-dimensional Hilbert space), the paper discretizes the real line into `M = 2^m` points and defines discretized Hermite states `|ψ_n⟩ ∈ ℂ^M` as the (normalized) vector of Hermite-function values at those grid points. The **Quantum Hermite Transform Problem** is then: given a dimension `N` and error tolerance `ε`, construct a quantum circuit `U` performing

```
Σ_n α_n |n⟩  ↦  Σ_n α_n |ψ_n⟩
```

to within additive error `ε`, for arbitrary input superpositions `Σ α_n |n⟩`.

### 3.2 Theorem 1 — exponential fast-forwarding of the QHO

> *For `N ≥ 1` and `t ∈ [−π, π]`, one can choose `M = Θ(N log N)` such that the evolution operator `e^{-iH̄t}` can be simulated within error `O(exp(−N/10))`, in the subspace spanned by the first `N` eigenvectors of the discretized QHO Hamiltonian `H̄`, using `O(log² N)` gates.*

This is the central result. It significantly improves on the best prior result (Somma, 2016), which only achieved a *sub-exponential* fast-forwarding cost of `~exp(√(log N))`. The technique is a succinct operator factorization (rather than the Trotter–Suzuki product-formula approach of the prior work):

```
e^{-iĤt} = exp(-i·tan(t/2)·p̂²/2) · exp(-i·sin(t)·x̂²/2) · exp(-i·tan(t/2)·p̂²/2)
```

This factorization is *exact* in the continuum (derivable via the Baker–Campbell–Hausdorff formula, exploiting the fact that the Lie algebra generated by `x̂²` and `p̂²` closes after only three elements: `x̂²`, `p̂²`, and the anticommutator `{x̂, p̂}`). The paper's substantial technical work is proving that this factorization survives the transition to a *discretized* version of `x` and `p` with only **doubly-exponentially small** error in the dimension `M`, within the low-energy subspace spanned by the first `N` eigenstates — despite the discretized `x̄` and `p̄` no longer exactly satisfying the canonical commutation relation `[x̄, p̄] = iI`.

Each of the three exponential terms is a **diagonal** unitary (in the position basis for the `x²` term, and in the momentum/Fourier basis for the `p²` terms), implementable with `O(log² M)` two-qubit gates via coherent arithmetic and phase kickback — and since there are only three (or, for the full angular range, five) such terms regardless of `t`, the total circuit size is independent of `t`.

### 3.3 Theorem 2/19 — the efficient QHT circuit

Building on Theorem 1, the paper's main result for the transform itself:

> *There exists a quantum circuit of complexity `O((log N + log(1/ε))³ × log(1/ε))` that performs an `ε`-approximate quantum Hermite transform of dimension `N`.*

This matches the QFT's efficiency class (polylogarithmic in the dimension). The construction proceeds in four steps for each basis state `|n⟩`:

1. Adjoin `m` ancilla qubits: `|n⟩ → |n⟩|0⟩^⊗m`
2. **Hermite state preparation**, conditioned on `|n⟩`: `|n⟩|0⟩^⊗m → |n⟩|ψ_n⟩`
3. **Uncomputation** of the register holding `n` (inferred back out from `|ψ_n⟩` via fast QPE, enabled by Theorem 1): `|n⟩|ψ_n⟩ → |0⟩^⊗m|ψ_n⟩`
4. Discard the now-empty ancillas: `|0⟩^⊗m|ψ_n⟩ → |ψ_n⟩`

Step 2 (state preparation) itself has two sub-steps: first prepare an efficiently-constructible *approximation* `|φ_n⟩` to the target Hermite state, using **Plancherel–Rotach asymptotics** (a classical result giving provably accurate closed-form approximations to Hermite functions), then use **fixed-point amplitude amplification**, with a custom "filtering" oracle built from fast QPE, to boost `|φ_n⟩` into the true `|ψ_n⟩` to arbitrary precision.

### 3.4 A note on what "N" means for the advantage claim

The paper is careful to flag an important subtlety: the polylogarithmic scaling is in `N`, the *dimension of the Hilbert space* being transformed. In the property-testing/learning applications (Section 4 below), the relevant advantage shows up already at `N = poly(n)` for `n` input variables — meaning the QHT circuit itself only contributes a **polynomial** improvement to the overall algorithm's circuit size for those specific tasks, even though the transform is asymptotically as efficient as the QFT. This is an honest, self-flagged limitation worth carrying forward into any claim about "the" advantage this result delivers.

---

## 4. Applications

### 4.1 Property testing and learning over the Gaussian distribution

The paper's primary application is **Hermite sampling**: given black-box query access to a function `f: ℝⁿ → ℝ`, construct a state whose amplitudes are proportional to `f`, then use the (inverse) QHT to sample from `f`'s Hermite spectrum, `Pr[k] = |f̂(k)|²` — the direct analogue of Fourier sampling, which underlies many of the most prominent quantum algorithms with proven query advantage.

Using Hermite sampling as a subroutine, the paper gives quantum algorithms — with complexity **independent of the number of input variables `n`** — for:

1. Testing whether `f` is close to a product of `k` sign functions
2. Testing whether `f` is close to being degree-`d`
3. Testing whether `f` is close to a Hermite polynomial
4. Agnostically learning "sparse concepts"

The paper proves genuine **quantum query advantage** (not just circuit-size convenience) for tasks (1) and (2) in the full version of the paper. It also gives a Gaussian analogue of the classical **Goldreich–Levin algorithm** (a foundational result in Boolean function analysis and cryptography, for finding the large Fourier coefficients of a function efficiently) — here, finding the large *Hermite* coefficients of a real-valued function.

### 4.2 Hamiltonian simulation

Because the Hermite functions are what physicists call **Fock states** for the QHO, and Fock states appear naturally in *many* quantum systems (not just literal harmonic oscillators), the QHT is expected to be broadly useful for Hamiltonian simulation whenever a system's natural Hamiltonian becomes sparse when expressed in the Fock/Hermite basis. The paper gives a concrete example: the **Jaynes–Cummings model**, describing a two-level atom interacting with a quantized electromagnetic field mode — in the Fock basis, its Hamiltonian collapses into a direct sum of simple 2×2 blocks, meaning the QHT could enable exponential fast-forwarding of this model's discretized version too.

### 4.3 Open problems (as listed by the authors)

- Can the QHT yield learning algorithms for Linear (or general Polynomial) Threshold Functions that beat known classical query lower bounds?
- Is there a large class of differential equations that can be efficiently simulated on a quantum computer using the QHT, while remaining hard classically?
- Can other quantum systems beyond the QHO be fast-forwarded using similar techniques?
- Are there other natural concept classes, sparse in the Fourier domain, that can be learned via the QHT (extending Klivans–O'Donnell–Servedio's classical results)?

---

## 5. Independent Verification: What We Built and Found

Rather than taking the paper's claims at face value, the companion repository `qiskit-quantum-hermite-transform` implements the algorithmic core of Theorem 1 directly in Qiskit and checks it numerically at a small but honest scale (`M = 16–32`, i.e., 4–5 qubits). Full methodology, code, and honesty notes about implementation scope are in the repo's READMEs; the headline results are reproduced here.

### 5.1 The fast-forwarding factorization, implemented as a real circuit

We implemented the paper's full **Algorithm 1** — including the extended 5-term factorization needed for `π/2 < |t| ≤ π` (the simple 3-term formula alone diverges as `t → π` because of the `tan(t/2)` term) — using Qiskit's `DiagonalGate` for each `exp(-i·c·x²)` phase step, and an explicit centered discrete-Fourier-transform `UnitaryGate` to move between the position and momentum bases.

![Fidelity of the fast-forwarding circuit vs. evolution time t, flat at 1.0 from t=0.1 to t=200](images/fidelity_vs_t.png)

**Result: fidelity to the exact time-evolved state stayed ≥ 0.999999 for every tested `t` from 0.1 to 200** (using a *single, fixed* 7-gate circuit, after reducing `t` modulo the oscillator's `2π` period) — a direct, checkable demonstration of fast-forwarding.

### 5.2 Catching a real bug: the importance of testing the full claimed range

An earlier version of this implementation used *only* the simple 3-term factorization for every `t` (after reduction into `(−π, π]`). This produced a fidelity sweep with values **randomly cratering to as low as 0.23** whenever the reduced `t` exceeded `π/2` in absolute value. The cause: `a(t) = tan(t/2)/2` is only numerically well-behaved for `|t| ≤ π/2` — exactly the precondition the paper states for Theorem 5, which we had initially implemented without its accompanying case split. Implementing the paper's full Algorithm 1 (the 5-term "doubled" factorization for the wider angular range) fixed this completely. This is worth flagging because it is exactly the kind of subtle, easy-to-miss precondition that separates "read the abstract and believe it" from "actually implement and verify it."

### 5.3 Concrete resource comparison against naive simulation

![Constant fast-forwarding circuit size vs. linearly growing Trotter step count, log scale](images/fastforward_vs_trotter.png)

To make the practical stakes concrete: reproducing the same accuracy with a naive first-order Trotter product-formula simulation would require a number of sequential circuit repetitions that grows with `t` — reaching **almost 12,000 steps** by `t = 200` in our estimate, against a constant 7-gate circuit for the fast-forwarding approach at any `t`.

### 5.4 The Hermite states themselves

![Six discretized Hermite states, showing the correct number of nodes for each energy level](images/hermite_state_gallery.png)

We independently verified (via direct diagonalization of the discretized QHO Hamiltonian, for `M = 32`) that the discretized Hermite states defined by the paper's Eq. 9 are eigenstates of that Hamiltonian to **machine precision**: overlap `1.00000000` and eigenvalue error below `2 × 10⁻¹³`, for all six of the lowest states tested — visibly reproducing the expected `n`-node structure of each energy level.

### 5.5 Hermite sampling, as an actual measured circuit

![Measured Hermite-spectrum sampling distribution closely matching the true spectral weights 0.36 and 0.64](images/hermite_sampling_histogram.png)

We constructed a test function `f = 0.6·ψ₀ + 0.8·ψ₂` (known spectral weights `0.36` and `0.64`), encoded it as a quantum state, applied the inverse-QHT unitary, and measured — on an actual (simulated) quantum circuit, 4096 shots — sampling probabilities of **0.3594 and 0.6406**, matching the true weights to three decimal places.

### 5.6 Honesty about what was and wasn't reproduced

In the interest of the same standard applied throughout my other portfolio repos: the paper's stated `O(log² M)` gate-count for each diagonal phase step relies on an explicit **coherent-arithmetic** sub-circuit construction (computing `x_j²` into an ancilla register, phase kickback, uncomputation) that we did not re-derive gate-by-gate — we implement the identical *unitary* directly via Qiskit's `DiagonalGate`, which is exact but doesn't itself demonstrate that specific low-level gate-count scaling. Similarly, our Hermite state preparation uses Qiskit's `initialize` (exact, but exponentially costly) rather than the paper's own polylogarithmic-depth Plancherel–Rotach-plus-amplitude-amplification construction. What *is* faithfully and exactly verified is the algorithmic structure responsible for fast-forwarding specifically — a fixed number of steps, independent of `t` — which is the paper's most distinctive and checkable claim.

---

## 6. Where the Quantum Hermite Transform Sits in the Broader Landscape

| Primitive | Basis it transforms into | Circuit complexity | Status |
|---|---|---|---|
| Quantum Fourier Transform (QFT) | Fourier / momentum basis | `O(n²)` or `O(n log n)` | Decades old; foundational to Shor's algorithm, QPE, and dozens more |
| Amplitude amplification / Grover | — (amplitude reweighting, not a basis change) | `O(√(1/p))` iterations | Decades old; foundational to Grover-family search |
| Quantum walks | — (graph-structured search/sampling) | Problem-dependent | Established; powers ISD and lattice-sieving speedups (see `Quantum_Advantage_Crypto_Survey.md`) |
| **Quantum Hermite Transform (QHT)** | **Hermite / Fock basis (Gaussian-natural)** | `O((log N + log 1/ε)³ log 1/ε)` | **Introduced June 2026**; this report's subject |

The QHT is explicitly positioned by its authors as filling a real gap: the number of independent primitives quantum algorithms are built from is small, and each new one — genuinely structurally distinct from what came before, in Ning Bao's words — expands the *range* of problems a quantum computer could plausibly help with, beyond the recurring handful of number-theoretic and search-structured problems the field has mostly explored so far.

---

## 7. Conclusion

The quantum Hermite transform is a rare thing in current quantum algorithms research: a genuinely new primitive, with a provable, checkable, non-asymptotic-hand-waving core claim, published and independently reproducible within the same year. Unlike most entries in the crypto-focused survey this report's companion repository set extends, the "quantum advantage" here needs no forward-looking fault-tolerant hardware to become interesting — its central fast-forwarding claim about the quantum harmonic oscillator is a statement that can be, and in this report was, verified directly against classical linear algebra on a laptop, and found to hold with fidelity indistinguishable from 1 across every tested case.

What remains genuinely open — and is explicitly flagged as such by the paper's own authors — is whether this new primitive finds applications as broad and enduring as the QFT's. The property-testing and Gaussian-learning results in Section 4.1 are a first proof of concept; the open problems in Section 4.3 chart what the authors themselves see as the more consequential next steps. That is exactly the kind of frontier worth tracking, and building on, going forward.

---

## References

- Jain, S., Iyer, V., Somma, R. D., Bao, N., Jordan, S. (2026). *Efficient Quantum Hermite Transform.* Proceedings of the 58th Annual ACM Symposium on Theory of Computing (STOC '26). https://doi.org/10.1145/3798129.3800772
- Plata, C. (2026, July 13). *Building out the Quantum Computing Toolkit.* Brookhaven National Laboratory Newsroom. https://www.bnl.gov/newsroom/news.php?a=223007
- Somma, R. D. (2016). *Quantum simulations of one dimensional quantum systems.* Quantum Information and Computation, 16(13&14), 1125–1168.
- Yoder, T. J., Low, G. H., Chuang, I. L. (2014). *Fixed-point quantum search with an optimal number of queries.* Physical Review Letters, 113, 210501.
- Kitaev, A. Yu. (1995). *Quantum measurements and the Abelian stabilizer problem.* arXiv:quant-ph/9511026.
- Klivans, A. R., O'Donnell, R., Servedio, R. A. (2008). *Learning Geometric Concepts via Gaussian Surface Area.* FOCS 2008.
- Jaynes, E. T., Cummings, F. W. (1963). *Comparison of quantum and semiclassical radiation theories with application to the beam maser.* Proceedings of the IEEE, 51(1), 89–109.
- My companion repositories (Qiskit implementations discussed in Section 5): `qiskit-quantum-hermite-transform`, and the broader portfolio at `github.com/kanchanbhagtaniofc`.
