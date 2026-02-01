# Product Requirements Document (PRD)

**Project Name:** LABS-QE-MTS-GPU
**Team Name:** Nghi Thoi Gia
**GitHub Repository:** [PASTE_YOUR_FORK_URL]

---

## 1. Team Roles & Responsibilities

| Role | Name | GitHub Handle | Discord Handle |
| :--- | :--- | :--- | :--- |
| **Project Lead** (Architect) | Nghi Thoi Gia | nghinghi2005 | huhwhathow|
| **GPU Acceleration PIC** (Builder) | Nghi Thoi Gia | nghinghi2005 | huhwhathow |
| **Quality Assurance PIC** (Verifier) | Nghi Thoi Gia| nghinghi2005| huhwhathow |
| **Technical Marketing PIC** (Storyteller) | Nghi Thoi Gia | nghinghi2005 | huhwhathow |

---

## 2. The Architecture
**Owner:** Project Lead

### Choice of Quantum Algorithm
* **Baseline (completed in Phase 1):** Digitized Counterdiabatic (CD) circuit from the tutorial, sampled to seed Memetic Tabu Search (QE-MTS).
* **Phase 2 Algorithm (planned):** QAOA-style ansatz for the LABS cost Hamiltonian (or a light VQE variant), used specifically as a *seed generator* (not necessarily a full solver).

### Motivation
* The CD circuit is attractive for gate efficiency, but Phase 2 requires trying a **different quantum strategy**. QAOA is a widely understood baseline with clear knobs (depth p, mixer choice, parameter transfer).
* We will evaluate whether QAOA samples bias toward lower LABS energies than random sampling for the same N, and whether that bias translates into faster / better MTS convergence.

### Literature Review (initial)
* **Scaling advantage with quantum-enhanced memetic tabu search for LABS** (authors as referenced in the tutorial): motivates the QE-MTS workflow and success metrics.
* **CUDA-Q QAOA reference docs / examples**: implementation guidance and backend selection.

---

## 3. The Acceleration Strategy
**Owner:** GPU Acceleration PIC

### Quantum Acceleration (CUDA-Q)
* Use CUDA-Q GPU backends on Brev (e.g., NVIDIA GPU-accelerated simulation) to increase shot throughput and support larger N / deeper circuits.
* Benchmark: time per `cudaq.sample` as a function of N, depth (p or trotter steps), and shots.

### Classical Acceleration (MTS)
* Primary target: accelerate repeated LABS energy evaluations.
* Strategy options (we will choose at least one):
  1. Vectorize and batch-evaluate autocorrelations for many candidate flips.
  2. GPU offload with `cupy` (drop-in numpy replacement) for batched energy computation.
  3. Parallelize independent tabu search runs / children generation.

### Hardware Targets
* **Dev:** qBraid CPU for correctness and small-N experiments.
* **Porting:** Brev L4 (cost-efficient) for early GPU acceleration.
* **Final benchmarks:** Brev A100 for the largest N we can validate.

---

## 4. The Verification Plan
**Owner:** Quality Assurance PIC

### Unit Testing Strategy
* **Framework:** `pytest` or `unittest` (Phase 2 deliverable includes `tests.py`).
* **AI Hallucination Guardrails:**
  - Require property tests for invariances before accepting AI-generated code.
  - For small N, compare to brute-force optimum where feasible.

### Core Correctness Checks
* **Symmetry:** Assert `energy(S) == energy(-S)` and `energy(S) == energy(reverse(S))`.
* **Ground truth:** For small N (e.g., N<=12), brute-force enumerate all sequences to compute the true optimum and compare.
* **Cross-checking:** Compare energies of quantum samples vs random samples and verify median/quantiles improve for the quantum distribution (on small N where CPU sim is feasible).

---

## 5. Execution Strategy & Success Metrics
**Owner:** Technical Marketing PIC

### Agentic Workflow
* Use AI assistant to implement kernels and refactors; QA PIC runs tests on every change.
* Use short iteration loops: implement → validate small-N → scale.

### Success Metrics
* **Metric 1:** For fixed N, QE-seeded MTS achieves lower median final population energy than random-seeded MTS.
* **Metric 2:** For fixed target energy, QE-seeded MTS reduces iterations/time-to-solution.
* **Metric 3:** Demonstrate GPU speedup for energy evaluation and/or quantum sampling.

### Visualization Plan
* Histogram overlays (random-seeded vs quantum-seeded final population energies).
* Time-to-solution vs N on CPU vs GPU.

---

## 6. Resource Management Plan
**Owner:** GPU Acceleration PIC

* Develop and validate on CPU before turning on GPUs.
* Use short benchmarking windows; shut down Brev instances between sessions.
* Keep a simple cost log (start/stop times and GPU type).