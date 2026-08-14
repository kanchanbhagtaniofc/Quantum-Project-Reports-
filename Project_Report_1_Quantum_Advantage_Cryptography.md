# TECHNICAL PROJECT REPORT

## Project Title
Quantum Advantage Against Cryptographic Hardness Assumptions: A Survey, Cross-Verification Framework, and Extension Roadmap

## Project Type
Independent, self-initiated technical research and development project

## Author
Kanchan Bhagtani

## Status
Phase 1 complete (literature synthesis and PoC-availability cross-reference); Phase 2 (circuit-level implementation and verification) in progress — see Section 3 below and the companion repository `qiskit-cryptanalysis-portfolio`

## Report Date
August 2026

---

## 1. Project Motivation and Background

My work centres on post-quantum and quantum-safe cryptography — RLWE-based
cryptosystem design, quantum-safe architecture for defence networks, and
hash-based signature schemes, among other published work. A recurring
practical question in that work is: for a given cryptographic hardness
assumption, what is the actual, current state of quantum attack
capability against it — not the popular-press version of the question,
but the specific one that matters for parameter-setting and migration
planning: which quantum algorithms give genuine (exponential, sub-
exponential, or merely quadratic) speedup against which assumption, at
what resource cost (qubits and circuit depth specifically), and — a
question I found was almost never asked directly in the literature I
reviewed — which of those algorithms has actually been demonstrated on
physical quantum hardware, versus which remains a purely asymptotic
cost estimate with no experimental evidence at any scale.

This project began as an attempt to answer that question systematically
for myself, grounded in Biasse, Bonnetain, Kirshanova, Schrottenloher and
Song's 2022 IET Information Security survey, *"Quantum Algorithms for
Attacking Hardness Assumptions in Classical and Post-Quantum
Cryptography,"* extended with developments since that paper's
publication (the 2022 classical break of SIDH/SIKE, the 2024 NIST PQC
finalization, and 2025 resource-estimate updates for factoring RSA-2048).

## 2. Objectives

1. Produce a single, structured reference mapping every major
   cryptographic hardness assumption (classical and post-quantum) to its
   best known classical algorithm, best known quantum algorithm, and — a
   distinguishing feature of this project relative to the source
   literature — the concrete, resource-optimized (low-qubit-count,
   low-circuit-depth) variant of that quantum algorithm where one exists.
2. Compile, as a novel cross-reference not present in any single source I
   reviewed, which of these algorithms has an actual experimental
   proof-of-concept on physical quantum hardware, and at what scale —
   distinguishing genuine hardware demonstrations from simulator-only
   results and from "compiled"/non-generalizable demonstrations (e.g. the
   well-known but frequently over-cited N=143 "Shor's algorithm" result,
   which does not generalize).
3. Use that reference as the foundation for a second, implementation-
   focused project phase: independently building and verifying, in
   Qiskit, the core primitives this survey identifies as most consequential.

## 3. Methodology and Scope

This is a literature synthesis and cross-verification project, not a
claim of new theoretical results in quantum algorithms. Its
contribution is in the structure and completeness of the synthesis: a
28-row master table spanning factoring/discrete-log (the Shor-type
exponential break), symmetric-key search (the Grover-type quadratic
speedup), lattice/code-based/isogeny post-quantum hardness assumptions
(the modest, often QRACM/QRAQM-contingent speedups), and — added in a
later revision of this project — the HHL/QAOA family of algorithms
sometimes proposed for cryptanalytic applications, evaluated with
appropriate skepticism given the dequantization results (Tang et al.)
that undercut several of their early flagship claims.

Phase 2 of this project, documented separately in the companion
repository `qiskit-cryptanalysis-portfolio` and its sibling repositories
(`qiskit-linear-algebra-optimization`, `qiskit-fundamentals-and-error-
mitigation`, `qiskit-quantum-hermite-transform`, `qiskit-primitive-
polynomial-generation`), extends this survey from literature synthesis
into independent circuit-level verification: implementing and testing,
from scratch rather than via library calls, the algorithms this survey
identifies as most relevant to cryptographic hardness assumptions —
including a genuine reconstruction of the Kuwakado-Morii structural
attack on the Even-Mansour cipher via Simon's algorithm, and a
qubit-recycled (semiclassical) variant of Shor's algorithm illustrating
the resource-optimization theme that runs through this whole project.

## 4. Key Findings (Summary)

The full findings are documented in the survey itself (reproduced in
full below this project report front-matter). In brief:

- Only the Shor-algorithm family (integer factoring, discrete log in all
  its variants) gives a genuine, unconditional exponential quantum
  speedup against a deployed cryptographic assumption.
- Symmetric-key primitives face, at most, the quadratic Grover-type
  speedup, and that speedup parallelizes poorly — a fact with direct
  bearing on realistic key-length migration planning.
- Post-quantum (lattice, code-based) hardness assumptions face only
  modest, often memory-model-contingent quantum speedups, several of
  which (per Albrecht et al.'s honest reassessment of lattice sieving)
  may not survive once realistic QRACM/QRAQM access costs are priced in.
- Isogeny-based schemes (SIDH/SIKE) were eliminated from consideration
  by a *classical*, not quantum, attack — a finding directly relevant to
  how migration risk should be weighted across different PQC families.
- Of the ~28 algorithm families surveyed, only a small handful (Shor's
  algorithm at toy scale, Grover's algorithm at toy scale, and — the most
  advanced entry — Simon's algorithm, recently demonstrated with a
  genuine measured exponential speedup on IBM hardware up to 58 qubits)
  have any real-hardware experimental evidence at all. Everything
  underlying current NIST PQC standards has none.

## 5. Extension Work Completed and Planned

**Completed** (Phase 2, in the companion Qiskit repositories):
- Working, independently-verified implementations of Grover's, Shor's,
  and Simon's algorithms, including the Even-Mansour cryptanalytic
  application and a resource-optimized (qubit-recycled) Shor variant.
- HHL and QAOA implemented from first principles (not via deprecated/
  removed library calls) and stress-tested against the dequantization
  caveat this survey raises.
- Quantum error correction (3-qubit bit-flip code) benchmarked under a
  realistic noise model, directly relevant to assessing how close current
  hardware is to the fault-tolerance threshold this survey's resource
  estimates implicitly assume.

**Planned / in progress:**
- A Grover-search resource estimate (qubit count and circuit depth)
  against a toy reduction of one of my own published cryptographic
  schemes, to close the loop between this survey's general framework and
  my own protocol-design work.
- Extending the real-hardware-execution work beyond simulator results —
  running at least one of the verified circuits on IBM hardware directly,
  to add a genuine hardware data point to the PoC-availability table this
  project already tracks for every other entry.
- A quantum-classical co-processing repository (in progress) applying
  the same resource-aware framework this survey uses to Qiskit Runtime
  primitives, circuit cutting, and variational (VQE/QAOA) workflows —
  extending the project's scope from "attacks on cryptographic hardness
  assumptions" to the broader quantum-classical co-processing techniques
  relevant to running any of this survey's algorithms at meaningful scale.

---

## Full Report

*(The complete survey — the master table of ~28 hardness assumptions,
the experimental proof-of-concept status table, the timeline of
post-2022 developments, and full references — follows below, unedited
from the standalone version of this report.)*

---
# Quantum Advantage Against Hardness Assumptions in Cryptography and Information Security

*A structured literature survey based on Biasse, Bonnetain, Kirshanova, Schrottenloher & Song, "Quantum Algorithms for Attacking Hardness Assumptions in Classical and Post-Quantum Cryptography" (IET Information Security, 2022), extended with post-2022 developments (SIKE/SIDH break, updated RSA/ECC resource counts, NIST PQC finalization) found via literature search, current to mid-2026.*

---

## 1. Scope and framing

"Quantum advantage" against a computational hardness assumption can mean several distinct things, and the table below tries to keep them separate for each problem:

- **Exponential advantage** — the problem moves from super-polynomial classical time to polynomial (or quasi-polynomial) quantum time. This is the Shor's-algorithm regime (Hidden Subgroup Problem family).
- **Polynomial/sub-exponential advantage in the exponent** — the problem stays exponential, but the exponent shrinks (e.g., Grover's quadratic speedup 2ⁿ → 2^(n/2); lattice sieving 2^0.292n → 2^0.257n).
- **No known advantage** — believed quantum-hard; used as the basis of post-quantum (PQC) designs.

Most of these problems are not NP-complete in the classical complexity-theory sense (factoring and discrete log are in NP∩coNP and not known/believed NP-complete; SVP variants, ISD/syndrome decoding, and generic search *are* NP-hard or NP-complete). I have included the true NP-complete/NP-hard generic combinatorial problems (SAT, subset-sum/knapsack, exact cover, graph problems) separately since Grover-type quadratic speedups apply generically to all of them, and included the crypto-specific hardness assumptions that are the real target of the source survey.

---

## 2. Master Table

**Legend:** CRACM = classical RAM; QRACM = quantumly-addressable classical memory; QRAQM = quantumly-addressable quantum memory; T = time; M/S = memory/space; n = security parameter/bit-length as defined per row.

**On the "Qiskit PoC available?" column:** this asks specifically whether a Qiskit (IBM's open-source SDK) implementation is publicly documented — official (Qiskit Textbook / IBM Quantum Learning / maintained `qiskit-algorithms` library) or community (published paper/GitHub repo). This is a *software/tooling* availability question, distinct from Section 6 below, which asks whether the algorithm has been *run on real physical qubits*. A "Yes" here does not imply the code was run on hardware at cryptographically meaningful scale — most Qiskit Textbook notebooks default to simulator execution, with real-device runs (where they exist) limited to the same toy scales documented in Section 6.

| # | Problem / Hardness Assumption | Domain | Classical best algorithm & complexity | Quantum algorithm (technique) | Best known asymptotic quantum complexity | Resource-optimized / low-qubit variant | Qiskit PoC available? | Status / notes |
|---|---|---|---|---|---|---|---|---|
| 1 | **Integer Factorization Problem (IFP)** — breaks RSA | Public-key (classical) | General Number Field Sieve (GNFS): sub-exponential `exp(O(n^{1/3} log^{2/3} n))` | **Shor's algorithm** (Abelian HSP via QFT / order-finding) | Polynomial: `O(n^3)` gate-count class (n=bit-length) | Gidney & Ekerå 2019: 20M noisy qubits, 8h; Chevignard–Fouque–Schrottenloher 2024 (approx. residue arithmetic): ~1730 logical qubits but ~2×10¹² Toffolis; **Gidney 2025**: <1M physical qubits, ~1 week, ~6.5×10⁹ Toffolis (100× Toffoli reduction over CFS24); "Pinnacle" QLDPC-code architecture (2026 preprint): <100k physical qubits (unverified) | **Yes** — official Qiskit Textbook / IBM Quantum Learning notebook (`ch-algorithms/shor.html`); toy instances only (N≤21, matching real-hardware ceiling) | RSA-2048 broken in principle by a large fault-tolerant machine; **no cryptographically relevant quantum computer exists yet (2026)**. NIST/NSA mandating PQC migration. |
| 2 | **Discrete Logarithm Problem (DLP)**, finite field / safe-prime groups — breaks DSA, classic DH | Public-key (classical) | Index calculus / NFS variant: sub-exponential | **Shor's DLP algorithm** (Abelian HSP, 2 control registers) | Polynomial, `~2×` circuit of order-finding | Ekerå–Håstad short-logarithm variant reduces multiplications 3/4–1/4×; Ekerå lattice-based post-processing trades qubits for classical runs | **No official demo** — no dedicated Qiskit tutorial for general finite-field DLP; adaptable in principle from the same QFT/QPE primitives used in the Shor notebook | Same qubit-recycling / semiclassical-QFT tricks as factoring apply |
| 3 | **Elliptic-Curve DLP (EC-DLP)** — breaks ECDSA, ECDH | Public-key (classical) | Pollard-rho type: `O(√p)` fully exponential (no sub-exponential attack known) | **Shor's DLP algorithm** on elliptic-curve group law | Polynomial in field size; group op. is the bottleneck | Proos–Zalka (2003) original circuits; Häner et al. 2020, Roetteler et al. 2017 optimized circuits; recent (2025) 2D-lattice-adder resource analyses reduce Toffoli/qubit counts further | **No official demo** — a few unmaintained community GitHub repos attempt small-curve EC point-addition circuits in Qiskit; not part of Qiskit Textbook/IBM Quantum Learning | EC-DLP is *more* quantum-vulnerable relatively (no known sub-exp classical attack, so quantum gain is "bigger" in relative terms) than RSA |
| 4 | **RSA via short discrete log (p+q)** | Public-key (classical) | N/A (specialized) | **Ekerå–Håstad algorithm** (short DLOG derivative of Shor) | Same asymptotic class as #1/#2 but fewer modular multiplications | Tunable tradeoff parameter *s*; lattice-based (simultaneous Diophantine) post-processing | **No** | Outperforms plain Shor for RSA/DH with structured exponents |
| 5 | **Hidden Subgroup Problem, general abelian G** | Foundational / number theory | Varies | **QFT-based HSP algorithm** | Polynomial in log\|G\| | — | **Yes (indirect)** — official Qiskit Textbook `Quantum Fourier Transform` and `Quantum Phase Estimation` notebooks implement the HSP-solving primitive generically | Underlies #1–#4 |
| 6 | **Continuous HSP over ℝᵐ / Pell's equation, unit group, class group, Principal Ideal Problem (PIP)** | Algebraic number theory (breaks some lattice trapdoor schemes, e.g. Soliloquy, some ideal-lattice schemes) | Sub-exponential classical (index-calculus style over number fields) | **Hallgren; Eisenträger–Hallgren–Kitaev–Song; Biasse–Song** (continuous HSP, Lipschitz oracle) | Polynomial (fixed-degree fields); poly for S-unit group, class group, PIP in arbitrary-degree fields | — | **No** | Breaks cryptosystems relying on hardness of PIP in cyclotomic rings (e.g. some early NTRU-like schemes, Soliloquy) |
| 7 | **Dihedral HSP** (non-abelian) | Foundational | Sub-exponential classical | **Kuperberg's sieve** (2004/2013) / Regev (poly-space variant) | `2^{O(√log N)}` sub-exponential time; poly space (Regev) | Sieve variants trade qubits ↔ classical post-processing (poly memory) | **Partial** — Qiskit Textbook has a `Hidden Shift Problem` chapter (related abelian hidden-shift primitive), but no dedicated Kuperberg's-sieve (dihedral HSP) implementation exists | Used for SIDH-adjacent hidden-shift reduction (#26 below) |
| 8 | **Unstructured search / preimage of Boolean function** (generic) | Foundational | `O(N)` | **Grover's algorithm** | `O(√N)` queries | Amplitude amplification generalizes to biased initial states; parallelization is *sub-optimal* (√R depth reduction costs √R more total work — worse than classical parallel search) | **Yes** — official Qiskit Textbook `Grover's Algorithm` notebook + maintained `qiskit-algorithms` `Grover` class with built-in oracles | Core primitive reused in nearly every row below |
| 9 | **Symmetric-key exhaustive search** (any block cipher, key size κ) — AES, etc. | Symmetric crypto | `O(2^κ)` | **Grover key search** | `O(2^{κ/2})` | AES-128 Grover circuit optimized to ≈ 2^80 quantum gates (Jaques et al. 2020, "Grover oracles for AES/LowMC"); NIST security-level definitions (Level 1/3/5 ≈ AES-128/192/256) assume 2^{κ/2} quantum cost | **Partial (community only)** — no official AES circuit in Qiskit; community Qiskit implementations exist for lightweight ciphers, e.g. "Grover on SIMON" (Almazrooie et al., arXiv:2004.10686), not full AES-128 | Doubling key length is the standard mitigation, but is *conservative* since Grover parallelizes poorly (total work grows as √R when depth shrinks by √R) |
| 10 | **Hash preimage search** | Symmetric crypto | `O(2ⁿ)` | **Grover** | `O(2^{n/2})` | same circuit-optimization literature as AES | **Partial (community only)** — toy Grover-oracle preimage notebooks for small/simplified hash functions circulate in community repos; no official Qiskit SHA implementation | Motivates doubling hash output length for long-term preimage resistance |
| 11 | **Hash collision search** | Symmetric crypto | `O(2^{n/2})` (birthday, Pollard-rho, poly memory); classical parallel tradeoff `T·S = 2^{n/2}` | **Brassard–Høyer–Tapp (BHT)** algorithm (Grover + classical precomputed table) | `O(2^{n/3})` time, **but requires `O(2^{n/3})` QRACM** | Time–memory tradeoff `T²·M = 2ⁿ`; below `M ≤ 2^{n/5}`, QRACM assumption droppable (Chailloux–Naya-Plasencia–Schrottenloher 2017) | **No** — no Qiskit implementation of BHT-style quantum collision search (would require QRACM not modeled in Qiskit's circuit abstraction) | Less-than-quadratic speedup in practice due to memory cost; classical parallel collision search (`T·S=2^{n/2}`) is *not* beaten by any known quantum algorithm when memory is priced like time |
| 12 | **Structured symmetric constructions** (Even–Mansour, FX-construction, 2-round constructions) — *superposition* query model | Symmetric crypto | Classically secure (no polynomial attack) | **Simon's algorithm** (period finding in (ℤ₂)ⁿ), also Shor/Kuperberg-based generalizations | Polynomial (`O(n)` queries) — **requires quantum superposition oracle access to the cipher** | "Offline Simon" (Bonnetain et al. 2019/2021): only `O(n)` superposition queries total, rest classical — same asymptotic exponent but removes need for online superposition oracle | **Yes** — official Qiskit Textbook `Simon's Algorithm` notebook (run on simulator *and* real device in the textbook itself); community "Grover on SIMON" paper (arXiv:2004.10686) implements a Simon-based attack on the SIMON block cipher in Qiskit | Practical relevance debated: superposition-query model requires attacker to run the cipher itself in superposition (a strong, often unrealistic, assumption); classical-query-only variants give real but only polynomial-factor speedups |
| 13 | **Element distinctness / generic k-list matching** | Foundational (subroutine for ISD, sieving, subset-sum) | `O(N)` (or `O(N log N)`) | **Ambainis quantum walk** | `O(N^{2/3})` queries (general: `O(N^{L/(L+1)})` for L-collision) | — | **Yes** — official Qiskit Textbook `Quantum Walk Search Algorithm` chapter (generic element-distinctness-style search); no cryptographic (ISD/sieving) application implemented | Basis of quantum-walk speedups for ISD, subset-sum, sieving below |
| 14 | **Shortest Vector Problem (SVP)** — lattice hardness underlying most lattice PQC (worst case) | Post-quantum (lattice) | Enumeration `2^{(1/8)n lg n + o(n lg n)}` (Albrecht et al. 2020, poly space); heuristic sieving `2^{0.292n+o(n)}` time, `2^{0.2075n+o(n)}` memory (BDGL near-neighbor sieve) | **Quantum enumeration** (Montanaro backtracking); **quantum sieving** (amplitude amplification / quantum walk on near-neighbor buckets) | Enumeration: `2^{(1/4e)n lg n + o(n lg n)}` (square-root of classical exponent, Aono et al. 2018); Sieving: best known `2^{0.257n+o(n)}` time & memory (AA on near-neighbor buckets, 2021–22); walk-based variant `2^{0.2571n+o(n)}` | CRACM-only sieve variant: `2^{0.292n}` T with classical memory only (no QRACM); QRACM sieve: `2^{0.0767n}` extra *memory* savings claimed in some variants; QRAQM walk-based: `2^{0.0496n}` memory | **No** | Quantum speedup for SVP sieving is *sub-quadratic* in practice once QRACM/QRAQM memory-access costs are priced realistically (Albrecht et al. 2020 "Estimating Quantum Speedups for Lattice Sieves" — casts doubt that any speedup survives once memory-access latency is honestly modeled) |
| 15 | **Learning With Errors (LWE)** — underlies ML-KEM/Kyber, most FHE | Post-quantum (lattice, average-case) | Reduces to SVP/BDD via BKZ with block size β(n); best classical BKZ-β solver = sieving `2^{0.292β}` | Same BKZ→SVP-oracle reduction, oracle replaced by quantum sieve/enumeration | `2^{0.257β}` (quantum sieve oracle) instead of `2^{0.292β}` classically | Same low-memory variants as row 14 apply to the inner SVP oracle | **No** | Regev's worst-case quantum reduction (SIVP→LWE) gives LWE a *quantum* worst-case hardness guarantee even though best attacks are still lattice-reduction based; "quantum-state LWE" variants (Grilo–Kerenidis–Zijlstra) are efficiently solvable in one formulation but remain hard in another (Brakerski et al.) |
| 16 | **Short Integer Solution (SIS)** — underlies ML-DSA/Dilithium, some hash-based & FHE schemes | Post-quantum (lattice, average-case) | Reduces to SVP on `L⊥_q(A)` via BKZ | Same quantum SVP-oracle substitution | `2^{0.2571β_SIS}` | Chen–Liu–Zhandry (2021): poly-time *quantum* algorithm for SIS in ℓ∞-norm for specific (non-cryptographic) parameter ranges | **No** | No broad non-lattice quantum attack known for cryptographically relevant SIS parameters |
| 17 | **Approximate Shortest/Closest Vector (worst-case, general lattices), CVP/BDD** | Foundational lattice theory | Kannan embedding reduces CVP/BDD → SVP in dimension n+1 | Same SVP quantum speedups apply via embedding | — | — | **No** | — |
| 18 | **Information Set Decoding (ISD) / syndrome decoding** — underlies Classic McEliece, BIKE, HQC | Post-quantum (code-based) | Prange (1962); Stern (1989); Finiasz–Sendrier (2009); representation-technique MMT (2011) `2^{0.054n}`-type exponents; BJMM (2012) further reduction | **Bernstein's quantum Prange** (Grover over permutations); **Kachigar–Tillich** quantum walk (quantum Stern/quantum-BJMM) | Quantum Prange: `O(√(binom(n,w)/binom(n−k,w)))`; Quantum walk Stern: exponent multiplies classical list size by `^{2/3}`; Quantum BJMM: list-size exponent `^{4/5}` power w/ square-root gain on outer permutation search | All quantum ISD variants require **QRACM** for the walk's auxiliary data structure; no known algorithm beats Prange asymptotically for **sparse-error** (w = o(n)) regime, classically or quantumly (Canto Torres–Sendrier 2016) — an open problem | **No** | Classic McEliece considered most conservative PQC KEM candidate largely *because* no structural (non-ISD) attack, classical or quantum, is known |
| 19 | **Isogeny walk / "main isogeny problem"** between elliptic curves | Post-quantum (isogeny) | Depends on setting (below) | Depends on setting | Depends on setting | — | **No** | Field mostly reset after 2022 SIDH break (see rows 24–26) |
| 20 | **Group action inversion on isomorphism classes** (CSIDH, "hard homogeneous spaces") | Post-quantum (isogeny, commutative group action) | `O(√\|G\|)` classical (meet-in-middle / claw variants) | **Dihedral-HSP reduction + Kuperberg's sieve** | `2^{O(√log\|G\|)}` quantum time, poly quantum memory | Bonnetain–Schrottenloher (2020) & Peikert (2020) concrete cost analyses of CSIDH parameters under this attack; both propose *larger* CSIDH parameter sets in response | **No** | CSIDH's original (2018) parameters were shown under-sized once concrete Kuperberg-sieve costs were computed; "CSIDH on the surface" and follow-ups propose safer sizes |
| 21 | **Small-degree isogeny problem** (fixed ℓᵏ-isogeny path) — was the basis of SIDH/SIKE | Post-quantum (isogeny), **now broken classically, see row 24** | Meet-in-the-middle claw finding, `O(p^{1/4})` classical time/`O(p^{1/4})` memory | **Grover search over walk-strings** (low memory); **Tani's quantum-walk claw-finding** (Jaques–Schanck 2019) (high memory, QRAQM) | Grover: `O(ℓ^{k/2})` poly-memory; Tani walk: `Õ(p^{1/6})` depth/width (but `Depth×Width = Õ(p^{1/3})`, *worse* than Grover's `Õ(p^{1/4})` DW-cost) | Depth/width/gate tradeoffs studied in Jaques–Schanck 2019, Adj et al. 2020 | **No** | Was NIST Round-3/4 alternate KEM candidate (SIKE) until classical break in 2022 (row 24) — quantum-algorithm analysis is now largely of historical/methodological interest |
| 22 | **Generic isogeny problem** (no bounded-degree promise) — collisions in Charles–Lauter–Goren hash | Post-quantum (isogeny) | `O(p^{1/2})` classical | **Delfs–Galbraith / Biasse–Jao–Sankar** hybrid: random walk to F_p-rational curve + group-action HSP step | `Õ(p^{1/4})` quantum time (bottlenecked by the Grover-searched random walk to an F_p-curve) | — | **No** | Provides genuine non-trivial quantum speedup even absent short-degree structure |
| 23 | **SIDH with torsion-point auxiliary data** (Problem 10 in source survey) | Post-quantum (isogeny) | **Broken classically**, see row 24 | Kutas et al. 2021 showed sub-case reduces to abelian hidden-shift problem, solvable in quantum sub-exponential time (when `N₂ > p·N₁⁴`) | Sub-exponential quantum | — | **No** | First hidden-shift application to SIDH-style torsion-data cryptanalysis (pre-dates and is superseded by row 24) |
| 24 | **[POST-2022 UPDATE] Castryck–Decru attack on SIDH/SIKE** | Post-quantum (isogeny) | **Classical polynomial-time key recovery** (Kani's reducibility criterion applied to torsion-point images), Aug 2022; extended by Maino–Martindale and Robert to fully general SIDH; breaks SIKEp434 in <1 hour, SIKEp751 in ~20h on a single core | N/A — broken classically, no quantum step needed | N/A | — | N/A — broken classically, no quantum circuit relevant | SIKE eliminated from NIST PQC standardization (was Round-4 alternate). M-SIDH, FESTA (variants attempting repair) also broken in polynomial time (Castryck–Vercauteren 2023) for overstretched/F_p-rational parameters. **Entire isogeny-KEM branch of PQC effectively removed**; CSIDH (group-action based, no torsion data) survives but needs the larger parameters noted in row 20 |
| 25 | **NP-complete / NP-hard combinatorial problems generically** (SAT, exact cover, graph coloring, Hamiltonian cycle, clique, vertex cover, TSP decision versions) | Complexity theory / generic security reductions | `O(2^n)` (or best known SAT-solver exponent, e.g. `1.3^n`-type for 3-SAT) | **Grover's search** over the exponential solution space; **Montanaro's quantum backtracking** for tree-structured search (SAT via DPLL-style backtracking) | Quadratic: `O(2^{n/2})` generic Grover; **quantum backtracking**: `O(√(#nodes)·poly(n))` vs. classical `O(#nodes)` for any backtracking algorithm with query access to the search tree | — | **Yes** — official `qiskit-algorithms` `Grover` tutorial solves 3-SAT instances directly from DIMACS-CNF via `PhaseOracleGate` (`07_grover_examples` notebook) | No super-polynomial (Shor-like) quantum algorithm is known or believed to exist for NP-complete problems; BBBV (1997) black-box lower bound shows Grover's `Θ(√N)` is optimal for unstructured search, so quadratic is the ceiling absent problem structure |
| 26 | **Subset-Sum / Knapsack problem** | NP-complete combinatorial (basis of knapsack cryptosystems, e.g., Merkle–Hellman — already classically broken by lattice reduction, unrelated to quantum) | Classical: Horowitz–Sahni meet-in-middle `O(2^{n/2})`; Howgrave-Graham–Joux (2010) `2^{0.337n}`; Becker–Coron–Joux (2011) `2^{0.291n}`; best classical to date ≈ `2^{0.283n}`-ish variants | **Quantum walk on Johnson-graph representations** (Bernstein–Jeffery–Lange–Meurer 2013; Helm–May 2018); simple **Grover/amplitude-amplification + phase-estimation** variants (Bonnetain–Schrottenloher-style / "Quantum Tree Generator" for 0-1 knapsack) | Best heuristic quantum exponent **below 0.25** (`2^{<0.25n}`, quantum-walk on BCJ representations, Springer 2013 chapter); simple Grover/QPE-based approach: `O(2^{0.5n})` (quadratic speedup only, but only `n+t+1` qubits) | QRACM required for the fast quantum-walk variant; the simple amplitude-amplification+phase-estimation approach uses only `n+t+1` qubits (poly, no exotic memory model) — practical near-term tradeoff | **Yes** — published open-source Qiskit subset-sum oracle library (Benoit, Schwartz & Cytron, arXiv:2410.01775, code on GitHub), optimized for qubit/gate count under Grover search | Illustrates same generic pattern as lattice sieving: quantum-walk gives sub-Grover exponent but needs QRACM; naive Grover gives clean quadratic speedup with minimal qubits |
| 27 | **Graph Isomorphism** | NP-intermediate (not known NP-complete) | Best classical: quasi-polynomial (Babai 2015/2016), `exp(polylog n)` | No known Shor-like efficient quantum algorithm despite GI's close relation to the (non-abelian) Hidden Subgroup Problem over the symmetric group `S_n` | Best quantum: no improvement over quasi-polynomial classical (reduction to HSP over `S_n` has resisted efficient quantum solution for >20 years) | — | N/A — no known quantum algorithm exists to implement | Famous **open problem**: HSP over `S_n` (needed for GI) is believed to require entangled/non-standard measurements ("strong Fourier sampling" provably insufficient, Moore–Russell–Schulman 2008), unlike the abelian case that powers Shor's algorithm |
| 28 | **Boolean/Simon-type period finding on structured symmetric primitives** (see also row 12) | Symmetric crypto | Classically secure | Simon's algorithm | Polynomial | "Offline Simon" reduces required superposition queries to O(n) | **Yes** — same official Qiskit Textbook `Simon's Algorithm` notebook as row 12 | Duplicate cross-reference to row 12 for completeness in NP/crypto crossover discussions |
| 29 | **HHL algorithm — general quantum linear-systems solver** (foundational primitive underlying rows 30–31) | Foundational / quantum linear algebra | Conjugate gradient / Gaussian elimination: `O(N·s·κ)` for an `N×N`, `s`-sparse, condition-number-`κ` system (or `O(N^3)` dense) | **Harrow–Hassidim–Lloyd (HHL) algorithm** (QPE + controlled rotation + uncomputation); improved via LCU (Childs–Kothari–Somma 2017) and Quantum Singular Value Transformation/qubitization (Gilyén et al. 2019) | Original HHL: `O(log(N)·s²κ²/ε)`; QSVT/qubitization-based: near-optimal `O(sκ·polylog(N/ε))` — **but output is a quantum state ∝ solution, not the classical solution vector**, and problem is BQP-complete in general | Toy circuit implementations (2×2, 4×4 systems) run on 4–7 qubits; QSVT-based reformulation reduces query complexity closer to the `O(κ)` lower bound | **Yes (historically), now external** — HHL shipped inside core Qiskit (Aqua / `qiskit-algorithms`) 2019–2021, was **removed from core Qiskit in 2021–2022** for being too narrow/misleading as a general "speedup," now maintained as the standalone `quantum_linear_solvers` package (github.com/anedumla/quantum_linear_solvers); official IBM Quantum Learning tutorial still exists | **Not itself an attack on any named cryptographic hardness assumption.** Its exponential speedup requires a sparse, well-conditioned matrix, an efficient state-preparation oracle for `b`, and interest only in a *function* of the solution — conditions rarely satisfied by cryptanalytic linear systems. Frequently over-cited as a "quantum cryptanalysis" tool in secondary literature; treat any such claim with the same skepticism applied to row 31 |
| 30 | **Quantum algebraic attack on symmetric ciphers via HHL / Boolean equation solving** (Chen–Gao framework) — targets stream ciphers (Trivium, Grain-128/128a), AES, SHA-3/Keccak, multivariate PKC | Symmetric crypto & MPKC / algebraic cryptanalysis | Gröbner-basis or SAT-based algebraic attacks: solving a general Boolean equation system is NP-hard; best classical solvers are exponential in the worst case | **Chen–Gao HHL-based Boolean equation solver** (arXiv:1712.06239, *J. Syst. Sci. Complex.* 2022): linearizes the nonlinear system via a Macaulay matrix and applies HHL; concrete instantiation for **Grain-128/128a** keystream-based key recovery (Chen et al., *Quantum Inf. Process.* 2021) | Polynomial in system size **and** in the condition number `κ` of the Macaulay matrix — but the authors themselves show `κ` is exponentially large for well-designed ciphers, so real-world cost collapses back to exponential; concrete Grain-128 cost: `O(2^21·N^{3.5}·κ²·e^κ/ε^{0.5})` (Grain-128a: `2^{21.5}`) | No low-qubit or resource-optimized variant published — purely asymptotic analysis, no circuit resource count given | **No** — theoretical/asymptotic papers only; no Qiskit or other circuit-level implementation published for any of the named ciphers | The papers' own headline conclusion is best read as a **negative result / design criterion**: AES, SHA-3/Keccak, Trivium, and standard MPKC schemes remain secure against this specific attack *precisely because* their associated condition numbers are large — the authors propose "keep the condition number large" as a post-quantum design principle rather than claiming a practical break |
| 31 | **Dequantization of HHL-style / QML linear-algebra speedups** (Tang et al.) — cross-cutting caveat on rows 29–30 | Foundational / quantum machine learning, complexity theory | N/A — this result *is* the classical side of the comparison | **Not a quantum algorithm** — a classical "quantum-inspired" algorithmic technique (ℓ₂-norm/length-squared sampling-and-query access) that matches HHL-style quantum runtimes up to polynomial factors for low-rank matrices | N/A (negative result for quantum advantage) | N/A | **N/A** — no PoC relevant; this is a classical complexity-theoretic result, not a circuit | Tang (2018/2019) dequantized the Kerenidis–Prakash quantum recommendation-systems algorithm; Chia–Gilyén–Li–Lin–Tang–Wang (STOC 2020) generalized this into a framework dequantizing *most* known QML linear-algebra speedups (PCA, supervised clustering, low-rank regression). Does **not** kill every HHL use case (genuinely sparse, high-rank, well-conditioned systems with real quantum state-prep advantage can still win), but is the standard caution against taking any HHL-based cryptanalysis claim (row 30) at face value |
| 32 | **QAOA / quantum annealing for combinatorial cryptanalysis** — SAT/MILP/QUBO-formulated algebraic, differential, or correlation attacks on lightweight and reduced-round ciphers | Symmetric crypto / combinatorial cryptanalysis | SAT-solver-based or MILP-based algebraic/differential/linear cryptanalysis (e.g., CryptoMiniSat, MILP differential-trail search); problem-specific, exponential worst case | **QAOA** (Farhi–Goldstone–Gutmann 2014) or **quantum annealing** (D-Wave) applied to a QUBO/Ising encoding of the cipher's attack equations | **No proven speedup** — QAOA carries no general worst-case guarantee beyond Grover-like quadratic bounds in restricted cases; empirical studies on lightweight ciphers (PRESENT, reduced-round SIMON/SPECK) show no consistent advantage over classical SAT/MILP solvers at tested scales | Small QUBO instances (few dozen logical qubits) run on D-Wave annealers; small QAOA circuits (~10–20 qubits) on gate-model devices for toy reduced-round ciphers | **Partial** — official `qiskit-optimization` / `qiskit-algorithms` `QAOA` class is generic (not crypto-specific); community papers have applied it to small cryptanalytic QUBO instances (e.g., reduced-round lightweight-cipher key recovery as MAX-SAT/QUBO), but there is no maintained Qiskit cryptanalysis toolkit | Included as the other major "quantum algorithm family" (besides HHL) regularly proposed for cryptanalysis in the literature; currently **no evidence of quantum advantage** for this application at any tested scale — status is closer to row 25's generic NP-complete-problem ceiling (at best quadratic, likely not even that in practice given NISQ-era noise) |

---

## 3. Notes on complexity classes represented

| Category | Representative rows | Nature of quantum advantage |
|---|---|---|
| Abelian Hidden Subgroup Problem (Shor-type) | 1–6 | **Exponential → polynomial.** The clean, unconditional quantum win. |
| Dihedral / non-abelian HSP | 7, 20, 23 | **Exponential → sub-exponential** (`2^{O(√n)}`), not polynomial. |
| Grover-type unstructured/generic search | 8–13, 25, 26 | **Quadratic speedup only** (provably optimal in the black-box model — BBBV bound). |
| Lattice/code/isogeny "hard" post-quantum problems | 14–18, 21–22 | **Modest constant-factor exponent shrinkage** (sieving 0.292→0.257 etc.), sometimes erased once realistic quantum-memory costs (QRACM/QRAQM) are priced in. |
| Structural quantum-only attacks (Simon-based) | 12, 28 | **Polynomial**, but contingent on an unrealistic superposition-query threat model for most practical purposes. |
| Believed quantum-immune / broken by classical means only | 24, 27 | No quantum advantage known (isogeny torsion attack is purely classical; Graph Isomorphism resists Shor-style attack entirely). |
| Linear-algebra / HHL-based algorithms and their optimization-based cousins | 29–32 | **Conditional / mostly illusory.** HHL's own exponential speedup requires sparsity + good conditioning + a weak (function-only) output requirement that real cryptanalytic problems rarely satisfy; its flagship early "wins" were subsequently dequantized (row 31). QAOA/annealing (row 32) has no proven asymptotic advantage at all for cryptanalysis. Included because both are the most commonly *mis-cited* "quantum threat" in secondary/popular literature, despite the weakest evidentiary basis of any category in this table. |

---

## 4. Timeline of key post-2022 developments not in the source survey

- **July–August 2022:** Castryck–Decru attack breaks SIDH/SIKE in classical polynomial time; Maino–Martindale and Robert generalize it to break SIDH completely regardless of starting curve. NIST removes SIKE from consideration.
- **2023:** Castryck–Vercauteren break M-SIDH and FESTA (proposed SIDH-repair variants) in polynomial time for F_p-rational or overstretched parameter choices.
- **August 2024:** NIST finalizes **FIPS 203 (ML-KEM**, ex-CRYSTALS-Kyber, lattice/MLWE-based KEM), **FIPS 204 (ML-DSA**, ex-Dilithium, MLWE/MSIS-based signatures), and **FIPS 205 (SLH-DSA**, ex-SPHINCS+, hash-based signatures, quantum security resting only on hash collision/preimage resistance i.e. rows 10–11 above).
- **March 2025:** NIST selects **HQC** (code-based, i.e. row 18's ISD hardness) as a structurally-independent backup KEM to ML-KEM; FIPS draft expected 2026, final ~2027.
- **2024–2025 (FN-DSA/FALCON):** standard drafted but not yet finalized as of this writing.
- **2024 (Chevignard–Fouque–Schrottenloher):** approximate-residue-arithmetic technique for Shor's algorithm cuts *logical qubit* count for RSA-2048 dramatically (~1730 qubits) at the cost of a huge (~2×10¹²) Toffoli-gate count.
- **July 2025 (Gidney):** combines CFS24 approximate arithmetic with yoked surface codes and magic-state cultivation to factor RSA-2048 with **<1 million *physical* qubits in about a week** — a 20× reduction from the 2019 Gidney–Ekerå estimate (20M qubits/8h), while cutting Toffoli count >100× relative to CFS24.
- **2025–2026 (various, e.g. "Pinnacle" QLDPC architecture):** further architecture-level (not algorithmic) proposals claim sub-100k-physical-qubit RSA-2048 factoring; these remain simulation/theory-only and unverified experimentally.
- **Lattice sieving:** post-2022 quantum-walk based sieves (Chailloux–Loyer 2021; Heiser 2021) push the best quantum heuristic sieving exponent to ≈0.257n, with continuing debate (Albrecht et al., "Estimating Quantum Speedups for Lattice Sieves") over whether *any* of this survives once QRACM/QRAQM access costs are honestly priced — i.e., whether lattice-based PQC needs to discount quantum attacks at all in practical parameter-setting.

---

## 5. Practical takeaways for parameter-setting / migration

1. **Only the abelian-HSP family (factoring, all standard flavors of DLP including EC-DLP) suffers a *true* exponential quantum break.** These are the schemes (RSA, (EC)DSA, (EC)DH) that must be replaced outright — not merely have their key sizes doubled.
2. **Symmetric primitives (AES, SHA-2/3, HMAC) need only a quadratic-speedup discount** (Grover): doubling key/output length is the standard, conservative fix, and is over-conservative in practice since Grover parallelizes badly.
3. **Lattice- and code-based PQC (ML-KEM, ML-DSA, HQC, Classic McEliece)** rely on problems (LWE/SIS/SVP, ISD) where the best quantum algorithms give only a small constant-factor exponent reduction, and even that reduction is contingent on expensive quantum-memory models (QRACM/QRAQM) whose physical realizability is itself an open architectural question.
4. **Isogeny-based PQC (SIDH/SIKE)** turned out to be broken by a *classical*, not quantum, attack — a reminder that "quantum-resistance" analysis must not crowd out classical cryptanalysis of new structures. CSIDH (a different isogeny construction, no torsion-point leakage) survives but requires enlarged parameters to resist the dihedral-HSP/Kuperberg-sieve quantum attack (row 20).
5. **NP-complete problems in general (SAT, subset-sum, TSP, graph coloring, etc.)** enjoy at best a quadratic Grover-type speedup and — per the Bennett–Bernstein–Brassard–Vazirani (1997) black-box lower bound — provably cannot do better *without exploiting problem-specific structure*. This is why subset-sum/knapsack-based or SAT-based cryptographic proposals are considered to have a comparatively small "quantum discount" relative to factoring/DLP-based ones.

---

## 6. Experimental Proof-of-Concept (PoC) Status — which of these have actually been run on real qubits?

This is the crucial reality-check missing from most resource-estimate papers: a circuit design and cost estimate is **not** the same as a physical demonstration. The table below adds that column. "PoC" here means a genuine execution on physical quantum hardware (superconducting, trapped-ion, photonic, NMR) — not a classical simulation of the circuit, and not a "compiled"/pre-factored circuit engineered around already knowing the answer (a widely-used but misleading shortcut in early Shor demos, flagged explicitly below).

| Row(s) in Master Table | Algorithm | Real-hardware PoC exists? | Largest/most notable demonstration to date | Caveats |
|---|---|---|---|---|
| 1 (Shor, factoring) | Shor's algorithm | **Yes — but only at toy scale, essentially unchanged for 20+ years** | N=15: NMR (2001, 7 qubits), photonic (2007, 2009), superconducting phase-qubit (2012). N=21: photonic qubit-recycling (2012, 2 photons/qudits), IBM superconducting (5–6 qubits, 2019/2021). Recent systematic study on 2025-era cloud hardware (IBM, IonQ, etc.) **still only reliably factors 15 and 21** despite orders-of-magnitude more qubits being available — the bottleneck is circuit depth/noise, not qubit count. Controversial outlier: N=143 (and later N=56,153) via *adiabatic* quantum computation with heavily "compiled" circuits — widely criticized as not implementing genuine Shor's algorithm, since the circuit is engineered using prior knowledge of the factors and doesn't generalize. | The real algorithm (uncompiled, with a genuine period-finding QFT on an unknown order) has **never exceeded N=21**. All "larger number" headlines involve either compilation tricks or non-scalable techniques. |
| 2–4 (Shor's DLP / EC-DLP / Ekerå–Håstad) | Shor DLP variants | **No** | Only classical simulation of Toffoli/gate networks (Roetteler et al. 2017; Proos–Zalka 2003) exists; no physical qubit execution of DLP/EC-DLP order-finding has been reported | Purely theoretical/simulated resource estimates to date |
| 6 (continuous HSP: Pell's eq., unit/class group, PIP) | Hallgren / EHKS / Biasse–Song | **No** | — | No known experimental realization at any scale |
| 7, 20 (Dihedral HSP / Kuperberg sieve, CSIDH attack) | Kuperberg's sieve | **No** | — | Cost analyses (Bonnetain–Schrottenloher, Peikert) are purely asymptotic/classical-simulation-based |
| 8–11 (Grover: generic search, symmetric-key search, preimage) | Grover's algorithm | **Yes, well-established but small-scale** | Robust multi-platform demonstrations at 2–6 qubits (search space up to ~64 elements); a widely cited 2024/2025 IBM 127-qubit study used the large chip mainly for noise/tomography characterization of a **3-qubit** logical Grover search, not a large search space. No cryptographically meaningful key-search (e.g., toy-AES) has been run beyond a handful of key bits. | Grover PoC scale is genuinely stuck around 3–6 qubits; nowhere close to demonstrating even a 10–16-bit brute-force key search |
| 12, 28 (Simon's algorithm — structured-cipher / Even–Mansour/FX attacks) | Simon's algorithm | **Yes — the most advanced crypto-relevant PoC in this whole table** | 2024–2025 IBM Quantum experiments (127-qubit Sherbrooke/Brisbane devices) demonstrated a **genuine, measured exponential quantum speedup** (not just circuit execution) for a restricted-Hamming-weight variant of Simon's problem, using circuits up to **58 qubits**, with dynamical-decoupling error suppression and measurement-error mitigation (Phys. Rev. X 2025). Separate cross-platform benchmarking (IBM superconducting vs. IonQ trapped-ion) exists using Simon's algorithm as a noise-diagnostic tool. Earliest PoC: 2014 photonic one-way (cluster-state) computer, 2-qubit logical version, 5 physical qubits. | This is currently the **only entry in the table with a laboratory-measured exponential speedup on a task with real algorithmic content** (rather than toy/compiled). Still far from attacking an actual cipher (FX/Even–Mansour) — it demonstrates Simon's *period-finding primitive*, not a full key-recovery attack on a real block cipher |
| 13 (element distinctness / k-list quantum walk — subroutine for ISD, sieving, subset-sum) | Ambainis quantum walk | **Partial** — generic discrete/continuous-time quantum walks demonstrated (NMR, photonic, superconducting, few qubits/vertices), but **no PoC of the cryptographic application** (no ISD, sieving, or hash-collision quantum walk has been run on real hardware) | Small graph/cycle quantum-walk demos (4–8 vertices) | Quantum-walk *hash functions* found in the literature are evaluated by classical simulation of collision rates, not by running on quantum hardware |
| 14–17 (Lattice SVP / LWE / SIS / CVP-BDD quantum sieving & enumeration) | Quantum sieving, quantum backtracking (Montanaro) | **No** | — | Entirely theoretical/asymptotic; requires QRACM/QRAQM models whose physical cost is itself unresolved (Albrecht et al. 2020) |
| 18 (ISD / code-based, Classic McEliece/BIKE/HQC) | Quantum Prange, quantum Stern, quantum BJMM | **No** | — | Theoretical only |
| 21–23 (Isogeny problems: small-degree, generic, SIDH torsion) | Grover-over-walks, Tani claw-finding, hidden-shift reduction | **No** (and now largely moot — see row 24, broken classically) | — | No quantum hardware run ever attempted these; SIKE was broken by classical mathematics before any quantum attack was relevant in practice |
| 25 (generic NP-complete: SAT, TSP, graph coloring, clique, etc.) | Grover / quantum backtracking | **Yes, toy-scale only** | Small SAT instances (a handful of variables) solved via Grover-oracle circuits on 3–6 qubit devices; quantum-annealing (D-Wave) demonstrations exist for some NP-hard problems but use a fundamentally different (adiabatic, non-gate-model, non-Grover) paradigm and their quantum speedup is disputed | No backtracking-tree PoC (Montanaro's algorithm) has been run on real hardware to my knowledge |
| 26 (subset-sum / knapsack) | Amplitude-amplification + phase-estimation subset-sum algorithm | **Yes, genuine small-scale PoC** | IBM's `ibmq_santiago` and `ibmq_bogota` devices: 4-element subset-sum instance, using `n+t+1` qubits, verified quadratic speedup over brute force at that tiny scale (Sci. China Inf. Sci., 2021). The 2025 "Quantum Tree Generator" 0-1 knapsack work is a classical-simulation/runtime-calculator study, not a hardware run. | Genuine but trivial instance size (4 items) — a classical laptop solves this instantly; value is purely as a proof of the amplitude-amplification circuit's correctness |
| 27 (Graph Isomorphism / non-abelian HSP over Sₙ) | — (no known efficient quantum algorithm) | **N/A** | — | Not applicable — no algorithm exists to demonstrate |
| 29 (HHL algorithm, general) | Harrow–Hassidim–Lloyd | **Yes, well-established, but tiny scale** | Numerous 2×2 and 4×4 linear-system demonstrations on IBM hardware since ~2018 (simulator-verified, some run on real devices); official Qiskit implementation existed in-core 2019–2021, now the external `quantum_linear_solvers` package | Every physical demonstration to date solves systems classical linear algebra solves instantaneously (2–4 variables); no path demonstrated yet to sparse, well-conditioned, cryptanalytically-relevant system sizes |
| 30 (HHL-based algebraic attack on ciphers — Chen–Gao) | Chen–Gao Boolean-equation HHL attack | **No** | — | Purely asymptotic papers; no circuit was ever built or run, even in simulation, for AES/Trivium/SHA-3/Grain-128 |
| 31 (Dequantization of HHL/QML) | Tang et al. | **N/A** | — | Classical algorithm/complexity-theory result; no quantum circuit to demonstrate |
| 32 (QAOA/annealing for cryptanalysis) | QAOA / D-Wave annealing | **Yes, toy scale** | Small (≤20-qubit gate-model, or tens-of-logical-qubit D-Wave) demonstrations against reduced-round lightweight ciphers in community papers | No demonstrated advantage over classical SAT/MILP solvers even at these toy scales; primarily feasibility studies, not speedup demonstrations |

### 6.1 Quick summary: PoC availability by category

| Category | Real-hardware PoC? | Best demonstrated scale |
|---|---|---|
| Shor's algorithm (factoring) | Yes, stagnant | N=21 (unchanged in practice since 2012 for genuine/uncompiled runs) |
| Shor's algorithm (DLP/EC-DLP) | No | — |
| Grover's algorithm (generic + symmetric-key search) | Yes | ~3–6 qubits / search space ≤ 64 |
| Simon's algorithm (structured-cipher attack primitive) | **Yes, most advanced in table** | Up to 58 qubits, measured exponential speedup (2024–25) |
| Quantum walk (element distinctness / ISD / sieving subroutine) | Generic walk yes; crypto application no | 4–8 vertex toy graphs |
| Subset-sum / knapsack | Yes, trivial scale | 4-element instance |
| Lattice SVP/LWE/SIS sieving & enumeration | **No** | — |
| Code-based ISD | **No** | — |
| Isogeny attacks (SIDH/CSIDH) | **No** | — (also now classically obsolete for SIDH) |
| Continuous HSP (Pell, class group, PIP) | **No** | — |
| Generic NP-complete (SAT, TSP, etc.) via Grover | Yes, toy | Few-variable instances, 3–6 qubits |
| Graph Isomorphism | N/A — no algorithm | — |
| HHL algorithm (general linear systems) | Yes, well-established | 2–4 variable toy systems |
| HHL-based algebraic attacks on ciphers (Chen–Gao) | No | — |
| QAOA/annealing for cryptanalysis | Yes, toy scale | ≤20 qubits / reduced-round ciphers, no advantage shown |

**Bottom line:** the only algorithms in this whole survey with a *bona fide, cryptography-adjacent, measured exponential-speedup demonstration* on real hardware today are variants of **Simon's algorithm** (row 12/28) — and even that demonstrates the underlying period-finding primitive on a structured oracle, not a full attack on a deployed cipher. Shor's algorithm has been physically run but is stuck at N=21 due to circuit-depth/noise limits, not qubit count. Grover's algorithm and the subset-sum amplitude-amplification circuit have clean small-scale PoCs but at instance sizes classical computers solve instantly. Every problem in the "post-quantum hardness assumption" bucket (lattice, code-based, isogeny) — i.e. exactly the problems PQC standards (ML-KEM/ML-DSA/HQC) are designed to resist — has **no experimental quantum attack whatsoever**, only asymptotic cost papers.

---

## 7. Primary sources consulted

- Biasse, Bonnetain, Kirshanova, Schrottenloher, Song. *Quantum Algorithms for Attacking Hardness Assumptions in Classical and Post-Quantum Cryptography.* IET Information Security, 2022 (uploaded document; primary structural backbone of this survey, Sections 3–10 in the original).
- Castryck, W., Decru, T. *An efficient key recovery attack on SIDH.* CRYPTO 2022 / ePrint 2022/975.
- Castryck, W., Vercauteren, F. *A polynomial-time attack on instances of M-SIDH and FESTA.* ePrint 2023.
- Gidney, C. *How to factor 2048 bit RSA integers with less than a million noisy qubits.* arXiv:2505.15917 (2025).
- Chevignard, Fouque, Schrottenloher. *Reducing the Number of Qubits in Quantum Factoring* (2024).
- NIST. FIPS 203, 204, 205 (Aug. 2024); HQC selection announcement (Mar. 2025).
- Bonnetain, X., Schrottenloher, A. *Quantum Security Analysis of CSIDH.* EUROCRYPT 2020; Peikert, C. *He Gives C-Sieves on the CSIDH.* EUROCRYPT 2020.
- Albrecht et al. *Estimating Quantum Speedups for Lattice Sieves.* ASIACRYPT 2020.
- Bernstein, Jeffery, Lange, Meurer. *Quantum Algorithms for the Subset-Sum Problem.* PQCrypto 2013.
- Moore, Russell, Schulman. *The Symmetric Group Defies Strong Fourier Sampling.* SIAM J. Comput. 2008 (Graph Isomorphism / non-abelian HSP hardness for quantum algorithms).
- Montanaro, A. *Quantum-walk speedup of backtracking algorithms.* Theory of Computing, 2018.
- Jaques, S., Schanck, J. *Quantum cryptanalysis in the RAM model: claw-finding attacks on SIKE.* CRYPTO 2019.

### Qiskit-specific sources (Master Table "Qiskit PoC available?" column)

- Qiskit/textbook (archived; superseded by IBM Quantum Learning). *Quantum Protocols and Quantum Algorithms* chapter: Deutsch–Jozsa, Bernstein–Vazirani, Simon's Algorithm, QFT, QPE, Shor's Algorithm, Grover's Algorithm, Quantum Counting, Quantum Walk Search Algorithm, Hidden Shift Problem notebooks. https://github.com/Qiskit/textbook/tree/main/notebooks/ch-algorithms
- Qiskit-community/qiskit-algorithms. `Grover` class and `07_grover_examples` tutorial (3-SAT via DIMACS-CNF `PhaseOracleGate`). https://qiskit-community.github.io/qiskit-algorithms/tutorials/07_grover_examples.html
- Koch, D., Wessing, L., Alsing, P. *Introduction to Coding Quantum Algorithms: A Tutorial Series Using Qiskit.* arXiv:1903.04359 (2019) — independent Qiskit tutorial series covering Simon's, Grover's, Shor's algorithms at gate level.
- Almazrooie, M. et al. *Grover on SIMON.* arXiv:2004.10686 — Qiskit-based Grover-meets-Simon key-recovery attack implementation on the lightweight SIMON block cipher.
- Benoit, A., Schwartz, S., Cytron, R. K. *Optimization of a Quantum Subset Sum Oracle.* arXiv:2410.01775 (2024) — open-source Qiskit subset-sum oracle library, published on GitHub.
- Chang, W.-L. et al. *Quantum algorithm and experimental demonstration for the subset sum problem.* Sci. China Inf. Sci. (2021) — Qiskit-based circuit run on `ibmq_santiago`/`ibmq_bogota`.

*(Full bibliographic detail for rows 1–23 is in the References section of the uploaded IET paper; only post-2022 additions are cited separately above.)*

### Experimental/PoC-specific sources (Section 6)

- Vandersypen, L. et al. *Experimental realization of Shor's quantum factoring algorithm using NMR.* Nature 414, 883–887 (2001) — N=15, 7-qubit NMR.
- Lanyon, B. et al. *Experimental demonstration of a compiled version of Shor's algorithm with quantum entanglement.* PRL 99, 250505 (2007); Lu, Browne, Yang, Pan. PRL 99, 250504 (2007); Politi, Matthews, O'Brien. *Shor's quantum factoring algorithm on a photonic chip.* Science 325, 1221 (2009) — photonic N=15 demonstrations.
- Martín-López, E. et al. *Experimental realisation of Shor's quantum factoring algorithm using qubit recycling.* Nature Photonics (2012), arXiv:1111.4147 — N=21, 2-photon qudit recycling.
- Skosana, U., Tame, M. *Demonstration of Shor's factoring algorithm for N=21 on IBM quantum processors.* Sci. Rep. 11, 16599 (2021).
- Amico, M., Saleem, Z., Kumph, M. *Experimental study of Shor's factoring algorithm using the IBM Q Experience.* Phys. Rev. A 100, 012305 (2019).
- *Practical Challenges in Executing Shor's Algorithm on Existing Quantum Platforms.* arXiv:2512.15330 (2025/2026) — systematic modern cloud-hardware study confirming stagnation at N=15/21.
- AbuGhanem, M. *Characterizing Grover search algorithm on large-scale superconducting quantum computers.* Sci. Rep. (2025), arXiv:2406.16018 — 3-qubit Grover on IBM 127-qubit devices.
- *A Scalable 5,6-Qubit Grover's Quantum Search Algorithm.* arXiv:2205.00117 (2022).
- *Feasibility and Limitations of Generalized Grover Search Algorithm-Based Quantum Asymmetric Cryptography.* MDPI Electronics (2025) — scalability degradation data for 2–4 qubit Grover-based crypto protocol.
- Watson, T. et al. / USC (Lidar group). *Demonstration of Algorithmic Quantum Speedup for an Abelian Hidden Subgroup Problem.* Phys. Rev. X 15, 021082 (2025), arXiv:2401.07934 — up to 58-qubit Simon's-problem exponential speedup on IBM Sherbrooke/Brisbane.
- *Simon's Algorithm in the NISQ Cloud.* arXiv:2406.11771 (2024) — IBM vs. IonQ cross-platform benchmarking via Simon's algorithm.
- Tame, M. et al. *Experimental Realization of a One-way Quantum Computer Algorithm Solving Simon's Problem.* arXiv:1410.3859 (2014) — earliest photonic PoC.
- *Demonstration of Exponential Quantum Speedup with Constant-Depth Compiled Circuits for Simon's Problem.* arXiv:2604.27457 (2026).
- Chang, W.-L. et al. and related. *Quantum algorithm and experimental demonstration for the subset sum problem.* Sci. China Inf. Sci. (2021) — `ibmq_santiago`/`ibmq_bogota` 4-element subset-sum PoC.
- Xu, N. et al. *Quantum factorization of 143 on a dipolar-coupling NMR system using the adiabatic algorithm.* PRL 108, 130501 (2012) — flagged as a "compiled"/non-generalizable demonstration, not genuine Shor's algorithm.
- Albrecht, M. et al. *Estimating Quantum Speedups for Lattice Sieves.* ASIACRYPT 2020 — argues no lattice-sieving PoC is imminent given QRACM/QRAQM cost uncertainty.

### HHL and related linear-algebra / optimization-based algorithms (rows 29–32)

- Harrow, A., Hassidim, A., Lloyd, S. *Quantum algorithm for linear systems of equations.* PRL 103, 150502 (2009) — original HHL paper.
- Childs, A., Kothari, R., Somma, R. *Quantum Algorithm for Systems of Linear Equations with Exponentially Improved Dependence on Precision.* SIAM J. Comput. 46(6) (2017) — LCU-based HHL improvement.
- Gilyén, A., Su, Y., Low, G. H., Wiebe, N. *Quantum singular value transformation and beyond: exponential improvements for quantum matrix arithmetic.* STOC 2019 — QSVT/qubitization-based near-optimal linear-solver framework.
- Chen, Y.-A., Gao, X.-S. *Quantum Algorithms for Boolean Equation Solving and Quantum Algebraic Attack on Cryptosystems.* arXiv:1712.06239; journal version *J. Syst. Sci. Complex.* 35, 373–412 (2022) — HHL-based algebraic attack framework applied to Trivium, AES, SHA-3/Keccak, and multivariate PKC.
- Chen, Y.-A. et al. *Quantum security of Grain-128/Grain-128a stream cipher against HHL algorithm.* Quantum Inf. Process. 20, 375 (2021) — concrete HHL-based key-recovery cost estimate for Grain-128/128a.
- Tang, E. *A quantum-inspired classical algorithm for recommendation systems.* STOC 2019; *Quantum-inspired classical algorithms for principal component analysis and supervised clustering.* arXiv:1811.00414 (2018) — dequantization of early flagship HHL-adjacent QML speedups.
- Chia, N.-H., Gilyén, A., Li, T., Lin, H.-H., Tang, E., Wang, C. *Sampling-based sublinear low-rank matrix arithmetic framework for dequantizing quantum machine learning.* STOC 2020 — general dequantization framework.
- Farhi, E., Goldstone, J., Gutmann, S. *A Quantum Approximate Optimization Algorithm.* arXiv:1411.4028 (2014) — original QAOA paper.
- `anedumla/quantum_linear_solvers` — external Qiskit-based HHL package continuing the functionality removed from core Qiskit in 2021–2022. https://github.com/anedumla/quantum_linear_solvers
