# TODO: deepbiomat-rs

> Atomic tasks. Open this repo only after P1, P2, P3 reach their respective stable tags AND Rust fluency is at the level documented in `README.md`.

**Cursor:** `v0.0.3` (gated; repo + all workspace crates renamed to `deepbiomat-*` for trademark hygiene — see [`CHANGELOG.md`](./CHANGELOG.md))

---

## v0.1.0 — Workspace Scaffold

- [ ] **0.1** Add LICENSE *(locked in v0.0.1: `MIT OR Apache-2.0` dual)*
  - [ ] 0.1.1 Add `LICENSE-MIT` file with canonical MIT text
  - [ ] 0.1.2 Add `LICENSE-APACHE` file with canonical Apache-2.0 text
  - [ ] 0.1.3 Add `NOTICE` file (start empty; required placeholder for Apache-2.0 attribution chain)
  - [ ] 0.1.4 Set `license = "MIT OR Apache-2.0"` in workspace `Cargo.toml` for shared metadata inheritance
  - [ ] 0.1.5 Add SPDX header `// SPDX-License-Identifier: MIT OR Apache-2.0` to every Rust source file
- [ ] **0.2** Toolchain pin
  - [ ] 0.2.1 `rust-toolchain.toml` — channel `stable`, components `rustfmt`, `clippy`, `rust-src`
  - [ ] 0.2.2 Decide MSRV (start at current stable - 2 minor versions)
  - [ ] 0.2.3 Document MSRV policy in `docs/msrv-policy.md`
- [ ] **0.3** Workspace setup
  - [ ] 0.3.1 Root `Cargo.toml` with `[workspace]` and shared profile config
  - [ ] 0.3.2 Crate skeletons under `crates/` per `ARCHITECTURE.md` §1 (each with empty `lib.rs`/`main.rs` + `Cargo.toml`)
  - [ ] 0.3.3 Workspace inheritance for `[package]` metadata (rust-version, license, repository, authors)
- [ ] **0.4** Tooling install + verification
  - [ ] 0.4.1 `cargo install cargo-skill` (per workflow standard)
  - [ ] 0.4.2 `cargo install cargo-nextest cargo-deny cargo-machete cargo-bloat cargo-msrv maturin`
  - [ ] 0.4.3 Verify each tool runs against the empty workspace
- [ ] **0.5** CI baseline (`.github/workflows/ci.yml`)
  - [ ] 0.5.1 Matrix: latest stable + MSRV; ubuntu, macos, windows
  - [ ] 0.5.2 Steps: fmt → clippy (deny warnings) → nextest → doc test → cargo-deny → cargo-machete → msrv verify
  - [ ] 0.5.3 Coverage step (`cargo llvm-cov`)
  - [ ] 0.5.4 Binary size check on `deepbiomat-cli` (warn now, enforce at v0.6.0)
- [ ] **0.6** Security baseline
  - [ ] 0.6.1 `deny.toml` with license allow/denylist
  - [ ] 0.6.2 `cargo audit` step in CI (or `cargo deny check advisories`)
  - [ ] 0.6.3 Dependabot config for cargo + GitHub Actions
  - [ ] 0.6.4 `CODEOWNERS`, `SECURITY.md`
- [ ] **0.7** Repo hygiene
  - [ ] 0.7.1 `.gitignore`, `.editorconfig`
  - [ ] 0.7.2 Pre-commit hook config (fmt + clippy on staged Rust files)
  - [ ] 0.7.3 Conventional Commits config + commit-msg hook
  - [ ] 0.7.4 `CHANGELOG.md` per Keep a Changelog
- [ ] **0.8** Tag `v0.1.0`

## v0.2.0 — `deepbiomat-core` Stable

- [ ] **1.1** Type design
  - [ ] 1.1.1 `Element` (newtype around atomic number with periodic-table data)
  - [ ] 1.1.2 `Composition` (HashMap-backed, normalized API)
  - [ ] 1.1.3 `Structure` (lattice + sites; minimal first)
  - [ ] 1.1.4 Unit newtypes (mass, length, energy) with safe conversions
- [ ] **1.2** `no_std` verification
  - [ ] 1.2.1 `#![no_std]` at crate root
  - [ ] 1.2.2 `extern crate alloc` where needed
  - [ ] 1.2.3 CI build target: `--no-default-features --target thumbv7em-none-eabihf` (or similar bare-metal target)
- [ ] **1.3** Feature flags
  - [ ] 1.3.1 `serde` feature (off by default)
  - [ ] 1.3.2 `std` feature (on by default; gates HashMap → BTreeMap fallback)
- [ ] **1.4** Tests
  - [ ] 1.4.1 Unit tests on every public type
  - [ ] 1.4.2 Property tests (`proptest` or `quickcheck`) for invariants
  - [ ] 1.4.3 Doctests on every public item
- [ ] **1.5** Docs
  - [ ] 1.5.1 Crate-level docs with example
  - [ ] 1.5.2 `cargo doc --no-deps` clean (no warnings)
- [ ] **1.6** Tag `v0.2.0`

## v0.3.0 — `deepbiomat-features` Subset Ported

- [ ] **2.1** Trait design
  - [ ] 2.1.1 `Featurizer` trait: `featurize`, `feature_labels`, `citations`
  - [ ] 2.1.2 Error type (`thiserror`-derived)
- [ ] **2.2** Port ≤ 3 featurizers from P2 chosen for inference-time value
  - [ ] 2.2.1 Featurizer 1
  - [ ] 2.2.2 Featurizer 2
  - [ ] 2.2.3 Featurizer 3
- [ ] **2.3** Equivalence tests
  - [ ] 2.3.1 Each Rust featurizer matches Python P2 output to `f32` precision on ≥ 10 hand-verified inputs
  - [ ] 2.3.2 Test fixtures shared across both repos (committed JSON)
- [ ] **2.4** Benchmarks
  - [ ] 2.4.1 Criterion bench per featurizer
  - [ ] 2.4.2 Baseline numbers recorded in `docs/benchmarks/v0.3.md`
- [ ] **2.5** Tag `v0.3.0`

## v0.4.0 — `deepbiomat-infer` Runs P1 Model

- [ ] **3.1** Backend decision
  - [ ] 3.1.1 Evaluate `tract` (ONNX inference) vs `candle` (native) for the task
  - [ ] 3.1.2 Decide; document in `docs/adr/0002-inference-backend.md`
- [ ] **3.2** Model loading
  - [ ] 3.2.1 `Model::load(path)` with format detection
  - [ ] 3.2.2 Provenance: model SHA-256 captured at load
- [ ] **3.3** Predict path
  - [ ] 3.3.1 `predict(features) -> Result<Output, InferError>`
  - [ ] 3.3.2 Batch variant
- [ ] **3.4** Equivalence with P1
  - [ ] 3.4.1 Run P1's model on a held-out sample in both Python and Rust
  - [ ] 3.4.2 Verify outputs agree to documented tolerance
- [ ] **3.5** Latency budget
  - [ ] 3.5.1 Declare per-task latency budget in `docs/benchmarks/`
  - [ ] 3.5.2 Criterion benches enforce in CI
- [ ] **3.6** Tag `v0.4.0`

## v0.5.0 — `deepbiomat-py` Bindings

- [ ] **4.1** PyO3 + maturin setup
  - [ ] 4.1.1 `pyproject.toml` for the Python package
  - [ ] 4.1.2 `#[pymodule]` exposing core/features/infer
  - [ ] 4.1.3 Type conversions (`Composition` ↔ Python dict, etc.)
- [ ] **4.2** Type stubs
  - [ ] 4.2.1 `.pyi` files committed
  - [ ] 4.2.2 `pyright` checks pass on stubs
- [ ] **4.3** Wheel build matrix
  - [ ] 4.3.1 manylinux + macos + windows
  - [ ] 4.3.2 Latest stable Python (and 1–2 prior minor versions)
  - [ ] 4.3.3 PyPI Trusted Publishing configured
- [ ] **4.4** TestPyPI dry-run
- [ ] **4.5** Python integration test (run pip-installed wheel against P3's data)
- [ ] **4.6** Tag `v0.5.0`

## v0.6.0 — `deepbiomat-cli` Published

- [ ] **5.1** CLI design
  - [ ] 5.1.1 `clap` subcommands: `featurize`, `predict`, `info`
  - [ ] 5.1.2 Config file support (TOML)
  - [ ] 5.1.3 Output formats: human, JSON, JSON Lines
- [ ] **5.2** Provenance
  - [ ] 5.2.1 Every output includes provenance block (versions, hashes)
- [ ] **5.3** Binary size enforcement
  - [ ] 5.3.1 Strip + LTO in release profile
  - [ ] 5.3.2 CI fails if `cargo bloat` reports > budget
- [ ] **5.4** Distribution
  - [ ] 5.4.1 Pre-built binaries attached to GitHub Release for ubuntu, macos, windows
  - [ ] 5.4.2 `cargo install deepbiomat-cli` works
- [ ] **5.5** Tag `v0.6.0`

## v1.0.0 — Public Release

- [ ] **6.1** Pre-publish hygiene
  - [ ] 6.1.1 `CHANGELOG.md` finalized
  - [ ] 6.1.2 README install instructions verified on fresh OS
  - [ ] 6.1.3 docs.rs preview build green for every crate
  - [ ] 6.1.4 All ADRs in `docs/adr/` committed
- [ ] **6.2** Publish
  - [ ] 6.2.1 Publish crates to crates.io in dependency order: core → features → infer → cli, then py
  - [ ] 6.2.2 Publish `deepbiomat-py` wheels to PyPI via maturin
  - [ ] 6.2.3 GitHub Release with CLI binaries attached
  - [ ] 6.2.4 Zenodo DOI minted
- [ ] **6.3** Discoverability
  - [ ] 6.3.1 README badges: crates.io, docs.rs, PyPI, CI, license, DOI
  - [ ] 6.3.2 Submit to `awesome-rust` and `awesome-materials-informatics` (PR upstream)
  - [ ] 6.3.3 Cross-link from P1, P2, P3 READMEs
- [ ] **6.4** Tag `v1.0.0`

---

## Backlog (Unscheduled)

- [ ] WASM target for `deepbiomat-core` + `deepbiomat-features` (browser-side featurization)
- [ ] GPU inference path behind feature flag (only if P3 demonstrates clear need)
- [ ] Additional featurizers ported from P2 as users request

## Open Questions

- _populated as workspace evolves_
