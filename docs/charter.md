# HestonMC-VG: Project Charter

## Project Identity

**HestonMC-VG** is a validation-grade Monte Carlo pricing engine for path-dependent equity derivatives under the Heston (1993) stochastic volatility model. The numeric core is templated on type, instantiating cleanly under `double`, `float`, and fixed-point (`ap_fixed`) precisions from a single source of truth. The project produces two primary documents: a methodology document with a fully decomposed end-to-end error budget for the engine itself, and an independent validation report applying the same standards to a published third-party reference implementation. The code exists to defend both documents.

## Why This Project Exists

Publicly available reference implementations of stochastic volatility Monte Carlo pricers are abundant, but very few are accompanied by the kind of explicit, term-by-term error decomposition that survives independent review. Most implementations report a price; some report a Monte Carlo standard error; almost none isolate and bound the discretization bias of the variance scheme, the quantization error introduced by reduced-precision arithmetic, the Gaussian-transform approximation error, or the RNG equidistribution properties — and almost none publish a parameter envelope outside which the implementation is explicitly not validated.

This project takes the opposite posture. The methodology document is the primary artifact and the code is the supporting evidence. Every numerical decision is named, every error term is bounded, every scope boundary is declared, and every result is reproducible from a fresh clone. The intent is to demonstrate what "validation-grade" means as a craft standard when applied to a small, well-understood model — not to advance the state of the art in Heston pricing, which is settled, but to advance the state of the art in *how the work is documented and defended*.

A second artifact, the validation report, applies the same standard to a published third-party reference implementation. Where the methodology document demonstrates how to *build* to validation grade, the validation report demonstrates how to *apply* validation grade to an implementation written by someone else — independently reproducing its results, probing its parameter envelope for failure modes, comparing its outputs against independent oracles, and producing a written assessment that either approves the implementation with findings or identifies specific reasons for rejection. The two documents together demonstrate both sides of the validation discipline: building correctly and assessing correctly.

## Function

Generate Monte Carlo price estimates for path-dependent equity derivatives under the Heston stochastic volatility model, with:

- Correct discretization of the coupled (S, v) SDE system, including explicit handling of the Feller-condition violation regime.
- Variance reduction techniques (antithetic variates at minimum; control variates and quasi-Monte Carlo as scope permits).
- A counter-based parallel-friendly RNG (Philox-4×32) with documented period and equidistribution properties.
- An inverse-CDF Gaussian transform suited to fixed-point implementation.
- A templated numeric core supporting `double`, `float`, and fixed-point (`ap_fixed`) instantiations from a single source of truth, with bit widths for the fixed-point case chosen by forward error analysis over the declared parameter envelope.
- Verification against both analytic (Heston Fourier-transform) and high-precision-MC (Broadie-Kaya or equivalent) oracles.
- A documented, term-by-term error budget bounding the total deviation from the true price.

In addition, the project produces an independent validation report on a published third-party Heston implementation (initially scoped to QuantLib's pricer, subject to confirmation at M1). The validation report uses the HestonMC-VG engine and its associated oracles as the independent reference, applies the same parameter envelope and error budget framework, identifies the third-party implementation's parameter envelope and observed failure modes, and produces a written assessment in the structure of an SR 11-7-style model validation report.

## Skill Stack Exercised

- **Stochastic calculus and numerical SDE methods**: Itô calculus, weak vs. strong convergence, Euler-Maruyama and Milstein discretization, the Feller condition and variance-process scheme selection (full truncation, reflection, Andersen QE), Broadie-Kaya exact simulation as a reference baseline.
- **Fourier methods for derivative pricing**: Heston characteristic function, Carr-Madan FFT, Lewis formulation; used to construct the analytic oracle.
- **Numerical analysis**: forward error analysis, range analysis, fixed-point quantization theory, Richardson extrapolation for empirical bias measurement, worst-case-corner reasoning.
- **Production C++**: header-only template design, numeric-type abstraction across `double`/`float`/`ap_fixed`, deterministic reproducibility across compilers and optimization levels, CMake-driven multi-target builds, modern test framework integration (Catch2 or GoogleTest).
- **Python orchestration**: oracle implementation, parameter sweep automation, regression test harness, methodology document plot generation.
- **Python/C++ interop via pybind11**: zero-copy NumPy buffer protocol, sub-microsecond call overhead, type-safe binding boundary, CMake integration.
- **Reproducible deployment**: Dockerized build and test environment targeting Linux x86_64.
- **Independent model validation practice**: structured review of a third-party numerical implementation against independent oracles, parameter envelope characterization, failure-mode probing, and written assessment in the form of an SR 11-7-style validation report.
- **Technical writing to validation standard**: derivation traceability, explicit assumption enumeration, defined-term discipline, failure-mode documentation.

## What It Demonstrates

1. **End-to-end mathematical ownership.** A single engineer derives the discretization scheme, bounds its bias, implements it in three numeric precisions, and defends the result against a closed-form oracle. There is no hand-off, no black box, no "I used a library."
2. **Hardware-engineering discipline applied to software numerics.** Fixed-point bit widths are chosen by analysis, not guessed. Quantization error is propagated forward through the kernel and bounded as an explicit term in the total error budget. Range analysis, forward error propagation, and worst-case-corner reasoning are applied to a software workload — disciplines that come from EE/hardware practice. The numeric core is structured so that fixed-point translation is trivial if a downstream project demands it; the discipline is exercised here even though no synthesis target consumes it.
3. **Validation-grade documentation discipline.** The methodology document could be picked up by an independent reviewer, followed end-to-end, and either approved or rejected on the strength of its own argument. The document explicitly enumerates assumptions, parameter envelope, error decomposition, verification procedure, and known limitations.
4. **Independent model validation practice.** The validation report demonstrates the activity itself, not just the standards: taking a third-party implementation, reproducing its results independently, probing its parameter envelope, comparing against independent oracles, and producing a structured written assessment with specific findings. This is the activity that model risk teams perform daily, demonstrated on a public reference implementation where the work product can be openly published.
5. **Reproducibility on a clean machine.** Linux x86_64, single-command Docker reproduction, no proprietary toolchains, no cloud dependencies, no vendor lock-in. Any reader can clone the repo and reproduce every result.
6. **Verification ladder rigor.** A multi-rung verification stack where each rung catches a distinct class of bug, with documented tolerances at each rung derived from the error budget rather than picked by feel.
7. **Production-grade software craft.** Templated numeric core, header-only design where appropriate, CMake build system, modern test framework, pybind11 bindings, CI on every commit, structured logging, configuration via environment, explicit failure-mode handling.

## Use Cases

- Pricing path-dependent equity derivatives under stochastic volatility, where validated accuracy matters more than peak throughput.
- Reference implementation against which a higher-throughput production pricer can be independently checked, with documented error bounds making the comparison meaningful.
- Inner pricing loop of a calibration routine that fits Heston parameters to observed option prices.
- Deterministic single-scenario revaluation of an exotic position when a hard real-time budget rules out a CPU under contention.
- Stress-test scenario revaluation where the shocks do not have closed-form responses.
- Published validation report serving as a public work sample of the model validation activity, of a kind that practicing model validators rarely produce in shareable form because their work product is normally confidential.
- Teaching reference for what a documented, defended numerical implementation looks like as a craft standard, and what an independent validation report against that standard looks like.

## Scope Evolution: FPGA Acceleration Removed

This project was originally scoped to include an FPGA acceleration phase targeting the AMD Artix-7 (Arty A7-100T) using Vitis HLS, with fixed-point arithmetic, on-board verification, and hardware-in-the-loop verification rungs. **That phase has been removed.** The reasoning, recorded for honesty and future reference:

1. **FPGA acceleration of Monte Carlo pricing is not a current production deployment pattern at meaningful scale.** Numerical pricing workloads have settled on CPU grids and, where throughput dominates, GPU compute. FPGA-for-MC-pricing is a research domain with strong academic results but limited production footprint. Building hardware acceleration into a *correctness reference* implementation does not align the artifact with how the underlying technique is actually deployed.

2. **The use case where FPGAs remain dominant is a different workload class.** FPGAs are the dominant production technology for ultra-low-latency market data ingestion, order book reconstruction, and tick-to-trade execution — network-layer state-machine and dataflow workloads, not numerical workloads. Combining a numerical pricing kernel with FPGA acceleration in the same artifact conflates two skill demonstrations that should be made independently.

3. **Schedule risk asymmetry.** The FPGA phase carried the highest variance in the project's critical path — HLS toolchain debugging, timing closure, on-board bringup, and hardware verification rungs are individually long-tailed and collectively the most likely place for the project to lose months. Removing the phase sharpens scope and concentrates effort on the artifact's actual value proposition.

4. **The throughput story was never the right story for this artifact.** The value proposition is methodology, correctness, and validation-grade documentation discipline. The FPGA phase was implicitly a *speed* story bolted onto a *correctness* artifact, which muddied both. Removing it sharpens the project's identity: this is a correctness showcase, full stop. Hardware acceleration of any kind belongs in projects whose primary value proposition is speed.

5. **Hardware-translation optionality is preserved.** The numeric core remains templated on `T` and is exercised under `ap_fixed` in host C++. Range analysis, forward error propagation, and explicit quantization budgeting are all part of the methodology. The fact that no synthesis target consumes the fixed-point output in this project does not reduce the value of doing the analysis — the analysis *is* the hardware-engineering contribution to the methodology, and the templated structure means a future project can add a synthesis target without rework of the numeric core.

## Target Architecture

### Deployment target: Linux x86_64

**Specifications**: 64-bit Linux on x86_64, Ubuntu LTS or RHEL/Rocky/Alma compatible. Modern C++ compiler (GCC 11+ or Clang 14+) with C++17 support. Python 3.10+. CMake 3.20+. Docker is the final reproducibility packaging, committed late in Phase 1; the build itself does not require Docker.

**Justification**:
- **Architectural alignment with where the work actually runs.** Quantitative pricing workloads run on Linux x86_64 in essentially every institutional environment that deploys them. The deployment target *is* the substrate, not a metaphor for it.
- **Reproducibility on commodity infrastructure**: any reader can clone the repo and reproduce every result on a machine that matches the target environment. No exotic hardware, no vendor lock-in, no proprietary toolchain license.
- **Toolchain stability and openness**: GCC, Clang, CMake, Catch2/GoogleTest, pybind11, NumPy, SciPy — all open source, all actively maintained, all standard.
- **Continuous portability verification**: GitHub Actions CI builds and tests on an Ubuntu LTS runner on every push from project start, catching portability drift between the maintainer's development host and the deployment target before it accumulates.
- **Single-command reproduction via Docker, late in Phase 1**: once the error budget is closed and the verification ladder is stable, a Dockerfile is committed alongside the source. `docker build` followed by `docker run` produces a container that runs the full verification ladder end-to-end with no host-side configuration. The Dockerfile is a *deliverable*, not a development tool — it captures the validated build environment after the project is substantively complete, rather than constraining the development loop along the way.

**Acknowledged limitations** (stated explicitly):
- This project does not target hardware acceleration (FPGA or GPU). The numeric core is *prepared* for fixed-point translation via the templated `ap_fixed` instantiation, but no synthesis target consumes it.
- Throughput is bounded by single-machine CPU performance. The artifact is a *correctness reference*, not a high-throughput pricing engine. A production deployment would either grid-parallelize across many nodes (trivially, since paths are independent) or use this engine as a challenger model against a higher-throughput vendor implementation.

### Software architecture: two-layer stack

**Layer 1 — Python control and oracle layer**
- Heston Fourier-transform analytic pricer (Carr-Madan or Lewis formulation), used as the oracle for the European-call limit and as a calibration target. May wrap QuantLib for cross-validation.
- High-precision Monte Carlo reference (Broadie-Kaya exact simulation or similar) for instrument-specific oracle where no closed form exists.
- Parameter envelope definition and management.
- Driver for parameter sweeps, convergence studies, and Richardson extrapolation experiments that empirically measure each error budget term.
- Regression test harness invoking the C++ core as a native extension module.
- Plot and table generation for the methodology document.

**Python ↔ C++ interface: pybind11.** The C++ numeric core is exposed to Python as a native extension module via pybind11, *not* as a subprocess invocation. Rationale: parameter sweeps, Richardson extrapolation studies, and calibration loops will call the C++ core thousands of times per session. Subprocess startup is 10–50 ms per call and JSON serialization of float arrays is both slow and lossy. pybind11 gives sub-microsecond call overhead, zero-copy NumPy buffer protocol for array passing, and type safety enforced at the binding boundary.

**Layer 2 — C++ numeric core (header-only, templated)**
- Single source of truth for: Philox-4×32 RNG, inverse-CDF Gaussian transform, Heston step kernel, payoff evaluation, statistical accumulators.
- Templated on numeric type `T` with a `numerics_traits<T>` abstraction layer covering `sqrt`, `exp`, `log`, `mul`, `add`. The same headers compile and execute correctly under three instantiations:
  - `double` — gold reference, used by the C++ test layer and as the bridge to the Python oracle.
  - `float` — intermediate, used to characterize sensitivity to precision.
  - `ap_fixed<W,I>` — fixed-point instantiation, with W and I selected by range analysis. Exercised in plain host C++ with the Vitis HLS headers on the include path; no synthesis required, no FPGA tooling required.
- This templating discipline is **architecturally non-negotiable**. It is what makes the verification ladder possible, it preserves optionality for future hardware acceleration without rework, and it forces a numeric discipline that pure-`double` code does not. Any function that cannot be templated this way is a function that needs to be rewritten before it ships.
- The fixed-point instantiation is a *first-class* part of the project, not a placeholder. Range analysis, bit-width selection, and quantization error propagation are exercised on real numbers and contribute the ε_quant term to the error budget.
- CMake build system with separate targets for: the C++ test binary (Catch2 or GoogleTest), the pybind11 extension module consumed by the Python layer, and the methodology-document data generators.

## Verification Ladder

Each rung catches a distinct class of bug. Tolerance at each rung is set by the relevant error budget terms, not chosen by feel.

| Rung | Comparison | Catches |
|------|------------|---------|
| 1 | Python oracle ↔ Python high-precision MC | Oracle implementation bugs |
| 2 | C++ `double` ↔ Python oracle | C++ algorithm bugs, RNG bugs, payoff bugs |
| 3 | C++ `float` ↔ C++ `double` | Single-precision sensitivity, catastrophic cancellation |
| 4 | C++ `ap_fixed<W,I>` ↔ C++ `double` | Fixed-point quantization within the predicted budget |

CI runs all four rungs on every commit on an Ubuntu LTS runner. Once the Dockerfile lands late in Phase 1, CI also runs the full ladder inside the published container as a final reproducibility check.

## Error Budget Framework

The methodology document's spine is a single defended inequality:

|Price_engine − Price_true| ≤ ε_MC + ε_disc + ε_var + ε_path + ε_quant + ε_invCDF + ε_RNG

with each term bounded as a function of implementation parameters over the declared envelope, and a target total below a stated threshold (e.g., 1bp of spot or 1% of price, whichever is larger).

| Term | Source | Bounding approach |
|------|--------|-------------------|
| ε_MC | Monte Carlo statistical error | σ/√N, measured per run |
| ε_disc | Asset-process discretization (Euler/Milstein) | Richardson extrapolation against Δt → 0 |
| ε_var | Variance-process scheme bias (full truncation / reflection / QE) | Comparison against Broadie-Kaya exact; literature bounds (Lord-Koekkoek-Van Dijk) |
| ε_path | Instrument-specific path-feature discretization | TBD per instrument choice |
| ε_quant | Fixed-point quantization | Forward error analysis through the step kernel |
| ε_invCDF | Inverse-CDF Gaussian approximation | Acklam / Wichura published bounds, verified empirically |
| ε_RNG | RNG bias and equidistribution | Philox period and TestU01 SmallCrush results |

The ε_path row is intentionally underspecified pending instrument selection (see Open Decisions).

## Development Sequence and Phase Gates

The project is divided into two sequential phases. Each phase has an explicit exit criterion that must be met before the next phase begins. The phases are deliberately scoped so that the **intellectual content of the project** (mathematical correctness, error budget, methodology document) is produced and defended in Phase 1, with Phase 0 as the upfront tooling investment that makes Phase 1 possible.

### Phase 0 — Infrastructure (before any numeric code)

Set up the scaffolding that everything else depends on. This is the kind of work that pays back across the entire project lifetime. The temptation to skip this and "just get something working" must be resisted; validation-grade work requires test infrastructure to exist *before* the code being tested.

- Repository structure, license, README skeleton.
- CMake build system with targets for test binary, pybind11 module, data generators.
- C++ test framework wired up (Catch2 or GoogleTest) with at least one passing dummy test.
- pybind11 binding skeleton with at least one trivial function callable from Python.
- Python project structure (pyproject.toml or equivalent), virtual environment, dependency lock.
- CI pipeline (GitHub Actions) running build + tests on every push against an Ubuntu LTS runner. CI is the always-on portability check against the deployment target.
- Methodology document skeleton committed with all section headers and TBD markers.
- Vitis HLS headers (specifically `ap_fixed.h` and dependencies) available on the include path so that `ap_fixed<W,I>` compiles in plain host C++. No Vivado or Vitis HLS toolchain installation required — only the headers.

**Development environment policy**: native development happens on the maintainer's host (currently macOS x86_64); the deployment target is Linux x86_64; portability is verified continuously via GitHub Actions on every push. An interactive Linux environment (VM or equivalent) is available for cases where CI feedback is insufficient. Docker is *not* part of the development loop in Phase 0 — it appears later in Phase 1 as a packaging and reproducibility deliverable, not as a development tool.

**Phase 0 Exit Criterion**: a fresh clone builds and tests successfully via CMake on both the native development host and the Ubuntu LTS CI runner; `import hestonmc` works in Python on both; all driven by CI on every push.

### Phase 1 — CPU implementation and full error budget

This is where the project's intellectual content is produced *and* where the project ends. Throughput is *not* a goal in this phase; correctness and defensibility are the only goals. The phase ends when the methodology document is substantively complete and the error budget is defended on real numbers, not estimates.

- Python oracle layer: Heston Fourier-transform pricer and the instrument-specific high-precision MC reference. Validated against published numerical examples and against QuantLib where applicable. (Verification rung 1.)
- C++ numeric core, templated on `T` from the first commit. Initial instantiation on `double`. Components: Philox-4×32 RNG, inverse-CDF Gaussian transform (Acklam or Wichura), Heston step kernel (full-truncation Euler as v1 baseline), payoff evaluation, statistical accumulators.
- pybind11 bindings exposing the C++ core to Python with NumPy buffer-protocol array passing.
- Verification rung 2: C++ `double` vs. Python oracle, within MC standard error.
- Verification rung 3: C++ `float` instantiation, characterizing single-precision sensitivity.
- Range analysis: instrument the `double` runs across the full parameter envelope to capture observed ranges of S, v, log S, √v, and intermediate products in the step kernel. Output: a documented range table that drives bit-width selection.
- Verification rung 4: C++ `ap_fixed<W,I>` instantiation in host code, with W and I selected by range analysis. Quantization error measured against `double` and shown to fall within the predicted ε_quant budget.
- Empirical measurement of every error budget term over the parameter envelope: ε_MC (trivial), ε_disc (Richardson extrapolation against Δt → 0), ε_var (vs. Broadie-Kaya or equivalent), ε_path (instrument-specific), ε_quant (rung 4), ε_invCDF (against high-precision CDF inverse), ε_RNG (TestU01 SmallCrush battery on Philox).
- Methodology document: all sections written with real numbers from real runs. Error budget inequality defended term by term over the declared envelope.
- Reproduction recipe: a Dockerfile committed late in Phase 1 (after the error budget is closed) that captures the full Linux x86_64 build and test environment. `docker build && docker run` produces a container that runs the full verification ladder end-to-end with no host-side configuration. Until the Dockerfile lands, the reproduction path is the documented CMake build sequence on Ubuntu LTS, mirrored by CI on every push.

**Phase 1 Exit Criterion (Project Exit Criterion)**: verification rungs 1–4 pass within the documented error budget for the chosen instrument over the declared parameter envelope. The templated C++ core builds cleanly under `double`, `float`, and `ap_fixed<W,I>` instantiations. The methodology document is complete. The Docker reproduction recipe works on a fresh Linux x86_64 box and the same recipe is what CI runs.

## Milestones

Milestones are checkpoints, not deadlines. Each milestone is a state of the repository that can be tagged and demonstrated.

| # | Milestone | Phase | Exit artifact |
|---|-----------|-------|---------------|
| M0 | Project charter committed | 0 | This document, in repo |
| M1 | Instrument selected, parameter envelope declared, validation target identified | 0 | `docs/envelope.md` committed; validation target named |
| M2 | Build, test, CI, pybind11 skeleton working end-to-end | 0 | Native build passes on dev host; Ubuntu LTS CI green on every push; `import hestonmc` works in Python on both |
| M3 | Python oracle layer validated against published references | 1 | Verification rung 1 passing in CI |
| M4 | C++ `double` core matches Python oracle within MC error | 1 | Verification rung 2 passing in CI |
| M5 | `float` precision sensitivity characterized and documented | 1 | Verification rung 3 passing; methodology section drafted |
| M6 | Range analysis complete, fixed-point widths chosen | 1 | `docs/range_analysis.md` committed |
| M7 | `ap_fixed` host build matches `double` within ε_quant budget | 1 | Verification rung 4 passing in CI |
| M8 | All error budget terms empirically measured and defended | 1 | Methodology document error-budget section complete with real numbers |
| M8.5 | Independent validation report on third-party implementation complete | 1 | `docs/validation_report.md` committed; structured assessment with findings |
| M8.7 | Dockerfile committed; full reproduction path containerized | 1 | `docker build && docker run` passes the full test suite on a clean Linux x86_64 host |
| M9 | **Project complete**: methodology document, validation report, reproduction recipe, all rungs passing | 1 | Repo tag `v1.0`; both documents complete; Docker reproduction recipe verified on a clean machine independent of the dev host |

Each milestone is small enough to ship and tag independently, which keeps the build cadence steady throughout the project lifetime rather than waiting for a single big-bang release.

## Deliverables

1. **Public repository** containing the full source tree (Python, C++), CMakeLists, Dockerfile, CI configuration, build instructions, and reproduction recipe.
2. **Methodology document** (`docs/methodology.md`), structured to validation standard: scope, assumptions, parameter envelope, model description, discretization derivation, error budget, verification procedure, results, known limitations, change log.
3. **Independent validation report** (`docs/validation_report.md`), applying the HestonMC-VG standards to a published third-party Heston implementation. Structured as an SR 11-7-style validation report: implementation under review, parameter envelope, methodology of the validation, oracle comparisons, observed behaviors and failure modes, findings, and reasoned assessment.
4. **Verification report** documenting each rung of the verification ladder with quantitative results against the error budget targets.
5. **Range analysis document** (`docs/range_analysis.md`) capturing observed numeric ranges across the parameter envelope and the resulting fixed-point bit-width selection with justification.
6. **Reproducibility recipe**: a single `docker build && docker run` invocation that takes a fresh clone to a passing test suite on any Linux x86_64 host with Docker installed. No host-side dependencies, no manual configuration, no platform-specific setup steps.

## Out of Scope (Explicit Non-Goals)

Validation documents are strengthened by stating what the work does *not* claim. This project explicitly does not address:

- **Calibration**: parameter envelope is declared, not calibrated to live market data.
- **American / Bermudan exercise**: no Longstaff-Schwartz or policy iteration. Path-dependent European-style payoffs only.
- **Multi-asset / basket products**: single-underlying only. Correlated multi-asset extension is a possible follow-on.
- **Rates, FX, credit, commodities**: equity-only parameter envelope.
- **Greeks via pathwise or likelihood-ratio methods**: bump-and-revalue only, if Greeks are produced at all.
- **Ultra-low-latency tick-to-trade execution**: this is a throughput-and-correctness artifact, not a tick-to-trade artifact.
- **Production deployment hardening**: no Kubernetes, no service mesh, no observability stack. The artifact is a reference implementation, not a productionized service.

## Open Decisions (To Be Resolved Before Implementation Begins)

1. **Instrument selection**. The choice of hero instrument drives the path-feature discretization (ε_path), determines the second oracle, and shapes the parameter envelope. Candidate families under consideration: discretely-monitored barriers, Asians, lookbacks, cliquets. Selection criteria: validation-document strength, oracle availability, fit with the overall scope.
2. **Parameter envelope**. To be declared explicitly once instrument is chosen: spot range, strike range, maturity range, (κ, θ, σ, ρ, v₀) ranges, instrument-specific parameters.
3. **Validation target selection**. The third-party implementation to be reviewed in the validation report. Default candidate: QuantLib's Heston pricer, on the basis that QuantLib is the most widely-used open-source quantitative finance library and a published validation report on it has the highest signal-to-effort ratio. Alternatives under consideration: Hudson and Thames' `mlfinlab` if it ships a Heston pricer, academic reference implementations from published papers, vendor evaluation versions if licensing permits redistribution of validation findings.
4. **Variance-process scheme priority**. Full truncation as the v1 baseline is near-certain. Whether Andersen QE is in v1 scope or deferred depends on implementation cost vs. error-budget improvement.
5. **Variance reduction scope**. Antithetic variates in v1 is near-certain. Control variates and randomized QMC are candidates for later iterations.

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Fixed-point widths inadequate for envelope; quantization error blows ε_quant budget | Medium | Medium | Range analysis before fixed-point instantiation; widen W and re-run; document the iteration honestly. Failure mode is bounded — host C++ iteration is sub-second. |
| Methodology document drifts from implementation over the project lifetime | High | Medium | Document plots and tables are generated by the same CI pipeline that runs the verification ladder; drift is detectable as test failures. |
| Range analysis misses an envelope corner where the variance process explodes | Medium | High | Explicit corner enumeration (Feller-violation regimes, near-zero variance, high vol-of-vol) as part of the parameter envelope declaration; verification rung 2 must pass at every declared corner. |
| Project scope expansion on a single technical detail | Medium | High | Hard scope discipline via the Out of Scope section; instrument selection deliberately constrained; v1 is a checkpoint, not a final form. Each milestone is small enough to ship in days, not weeks. |
| Validation-grade documentation expectations escalate as the project proceeds | Medium | Low-Medium | The methodology document skeleton is committed at M0 with all sections present as TBD. Filling in TBDs is a bounded activity; adding new top-level sections requires explicit decision. |
| Python oracle implementation has subtle bugs that propagate undetected through the verification ladder | Low | High | Rung 1 cross-validates the Python oracle against published numerical results from the Heston (1993) and Albrecher et al. (2007) papers, *and* against QuantLib where applicable. Two independent oracles. |
| Validation report uncovers a real defect in the third-party implementation | Medium | Low (positive outcome with process implications) | Treat as a successful validation finding, not a setback. Follow responsible disclosure practice: notify the maintainer privately first, allow reasonable time for response or fix, then publish the validation report with the finding documented. The finding *strengthens* the validation report's credibility — a report that finds nothing reads as ceremonial; a report that finds something real reads as actual review work. |
| Validation target (e.g., QuantLib) changes between when validation is performed and when report is published | Medium | Low | Pin the validation to a specific tagged release of the target implementation; cite the exact commit hash; note explicitly in the report that findings apply to that version. |

## Success Criteria

The project is complete when:

1. The verification ladder passes end-to-end through all four rungs within the error budget targets, and the CI pipeline runs the full ladder on every commit on an Ubuntu LTS runner.
2. The methodology document is complete, internally consistent, and could plausibly be handed to an independent reviewer without embarrassment.
3. The validation report on the third-party reference implementation is complete and committed, structured as an SR 11-7-style assessment with explicit findings and a reasoned conclusion.
4. The repository builds from a fresh clone on a clean Linux x86_64 box via the documented `docker build && docker run` reproduction recipe, with no host-side configuration.
5. The range analysis document is complete, the fixed-point widths are justified, and the ε_quant budget is defended with measured numbers from the `ap_fixed` host build.

The project is **not** complete when "the code works." It is complete when **both documents — the methodology document and the validation report — are defensible**.
