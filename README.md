# deepbiomat-rs

> **P4 of 4. Flagship.** Rust workspace productizing the patterns from P1–P3 into a narrowly-scoped, offline-first, constrained-machine-friendly library. Python bindings via PyO3/maturin.

**Status:** `v0.0.3` (pre-scaffold; repo + all workspace crates renamed to `deepbiomat-*` for trademark hygiene — see [`CHANGELOG.md`](./CHANGELOG.md))
**Repo path:** [`deepbiomat/deepbiomat-rs`](https://github.com/deepbiomat/deepbiomat-rs) (when created)
**License:** `MIT OR Apache-2.0` dual (locked 2026-04-21 — Rust ecosystem standard; downstream users select either at use time; preserves both max-compatibility and patent-grant optionality).
**Stack:** Rust (latest stable), Cargo workspace, PyO3 + maturin for Python bindings, optional `no_std` core. `cargo install cargo-skill` for skill discovery.

---

## Goal

Take the most useful patterns from P1 (model loading + inference) and P2 (featurization) and ship them as a small Rust library that:

- runs offline,
- runs on constrained hardware (laptop CPU, edge device, no GPU required for inference),
- exposes a clean Python API for users coming from the Python ecosystem,
- is maintainable solo — one workspace, ≤ 5 crates.

This is the project with your fingerprint. It is not a toy; it should be the thing you would install in your own workflow.

## Why Rust + Why Now

- **Determinism + portability:** Rust binaries run anywhere, no Python interpreter assumed.
- **Memory safety + performance:** matters for inference loops on constrained devices.
- **Single static binary:** easy distribution to environments where Python deployment is painful.
- **Mature interop:** PyO3 + maturin make Python bindings idiomatic.

The Python ecosystem (P1–P3) remains the right place to do research. Rust is the right place to ship inference and feature pipelines that need to live somewhere other than a notebook.

## Non-Goals

- **Not** a replacement for `pymatgen` / `matminer` / `DeepChem`.
- **Not** a training framework — Rust handles inference + featurization; training stays in Python.
- **Not** a from-scratch ML framework — use existing crates (`candle`, `burn`, `tract`) for tensor ops.
- **Not** a kitchen sink. ≤ 5 crates total. If a sixth is needed, the design is wrong.

## Workspace Shape (≤ 5 crates)

| Crate | Purpose | `no_std`? |
|---|---|---|
| `deepbiomat-core` | Core types: `Composition`, `Structure`, units. Zero deps. | yes |
| `deepbiomat-features` | Featurizers, port of P2. | no (uses `ndarray`) |
| `deepbiomat-infer` | Model inference (load + predict). | no (uses `tract` or `candle`) |
| `deepbiomat-py` | PyO3 bindings re-exporting the above. | n/a |
| `deepbiomat-cli` | Optional thin CLI for one-shot inference. | n/a |

## Constraints

- **MSRV:** declared at `v0.1.0`, bumped only with minor version.
- **Dependency budget:** each crate's direct deps documented with rationale in `Cargo.toml` comments.
- **Binary size budget:** CLI binary ≤ 20 MB stripped (declared at `v0.1.0`, enforced in CI).
- **Inference latency budget:** declared per task at `v0.4.0`, regression-tested in CI.
- **Offline-first:** no network calls in default code paths. Online features are opt-in via a `network` feature flag.

## Versioning

- `v0.1.0` — workspace scaffold, crate boundaries locked
- `v0.2.0` — `deepbiomat-core` types stable
- `v0.3.0` — `deepbiomat-features` ports a useful subset of P2
- `v0.4.0` — `deepbiomat-infer` runs P1's reproduced model end-to-end
- `v0.5.0` — `deepbiomat-py` Python bindings, install from PyPI via maturin
- `v0.6.0` — `deepbiomat-cli` published
- `v1.0.0` — crates.io publish (all crates) + PyPI publish (bindings) + docs.rs site green

## Prerequisites

- P1 ≥ `v1.0.0`, P2 ≥ `v1.0.0`, P3 ≥ `v0.5.0`
- Rust fluency to roughly the level of *Programming Rust* (Blandy/Orendorff/Tindall) chapters 1–17
- Familiarity with PyO3 + maturin

## Recommended Tooling

```
cargo install cargo-skill        # per workflow standard
cargo install cargo-nextest      # faster test runner
cargo install cargo-deny         # license + advisory + dupe checks
cargo install cargo-machete      # unused dep detection
cargo install cargo-bloat        # binary size analysis
cargo install cargo-msrv         # MSRV verification
cargo install maturin            # Python bindings build
```

## How to Cite

Will be added at `v1.0.0` with crates.io listing + Zenodo DOI.
