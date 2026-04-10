# hestonmc-vg

A validation-grade Monte Carlo pricing engine for path-dependent equity derivatives under the Heston (1993) stochastic volatility model. The work targets a fully decomposed end-to-end error budget — Monte Carlo statistical error, time-discretization bias, variance-process scheme bias, instrument-specific path-feature error, fixed-point quantization error, inverse-CDF approximation error, and RNG bias — with each term bounded as a function of implementation parameters over a declared envelope. The numeric core is templated on type, instantiating cleanly under `double`, `float`, and fixed-point (`ap_fixed`) precisions from a single source of truth. The deliverable is the methodology document; the code exists to defend it.

## Project state

Currently at **M0**: project charter committed. No implementation has begun. The full charter, including buyer positioning, scope evolution rationale, architecture, error budget framework, milestone definitions, and risk register, lives at [`docs/charter.md`](docs/charter.md).

The next gating decision is **M1**: instrument selection and parameter envelope declaration. Implementation work begins after M1 closes.

## Roadmap

| # | Milestone | State |
|---|-----------|-------|
| M0 | Project charter committed | ✓ |
| M1 | Instrument selected, parameter envelope declared | next |
| M2 | Build, test, CI, pybind11 skeleton working end-to-end | |
| M3 | Python oracle layer validated against published references | |
| M4 | C++ `double` core matches Python oracle within MC error | |
| M5 | `float` precision sensitivity characterized | |
| M6 | Range analysis complete, fixed-point widths chosen | |
| M7 | `ap_fixed` host build matches `double` within ε_quant budget | |
| M8 | All error budget terms empirically measured and defended | |
| M9 | Methodology document complete; reproduction recipe verified | |

See [`docs/charter.md`](docs/charter.md) for milestone exit criteria.

## License

Code: Apache License 2.0 — see [`LICENSE`](LICENSE).
Documentation: Apache License 2.0 unless otherwise noted at the top of the file.

## Author

ElectricQuant
