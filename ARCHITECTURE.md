# Architecture: deepbiomat-rs

## 1. Workspace Layout

```
deepbiomat-rs/
├── README.md
├── ARCHITECTURE.md
├── TODO.md
├── LICENSE-MIT              ← MIT
├── LICENSE-APACHE
├── NOTICE                   ← Apache-2.0 attributions if any
├── CHANGELOG.md
├── Cargo.toml               ← workspace root
├── Cargo.lock               ← committed (library workspace; lockfile useful for CI)
├── rust-toolchain.toml      ← pin channel + components
├── deny.toml                ← cargo-deny config
├── .cargo/
│   └── config.toml
├── .github/
│   └── workflows/
│       ├── ci.yml           ← fmt + clippy + test + cargo-deny + msrv
│       ├── docs.yml         ← docs.rs preview build
│       └── release.yml      ← crates.io + PyPI on tags
├── docs/
│   ├── adr/                 ← architecture decision records
│   ├── msrv-policy.md
│   ├── dependency-policy.md
│   └── benchmarks/
├── crates/
│   ├── deepbiomat-core/
│   │   ├── Cargo.toml
│   │   └── src/
│   ├── deepbiomat-features/
│   │   ├── Cargo.toml
│   │   └── src/
│   ├── deepbiomat-infer/
│   │   ├── Cargo.toml
│   │   └── src/
│   ├── deepbiomat-py/
│   │   ├── Cargo.toml
│   │   ├── pyproject.toml   ← maturin
│   │   └── src/
│   └── deepbiomat-cli/
│       ├── Cargo.toml
│       └── src/
├── examples/                ← integration examples per crate
├── benches/                 ← criterion benchmarks
└── xtask/                   ← optional: in-repo task runner
```

## 2. Crate Boundaries

```
                ┌────────────────────────────────────┐
                │           deepbiomat-cli               │
                │     (thin, delegates everything)   │
                └─────────────┬──────────────────────┘
                              │
                              ▼
                ┌────────────────────────────────────┐
                │           deepbiomat-py                │
                │       (PyO3 wrappers, no logic)    │
                └─────┬───────────────┬──────────────┘
                      │               │
                      ▼               ▼
       ┌──────────────────────┐  ┌──────────────────────┐
       │   deepbiomat-features    │  │    deepbiomat-infer      │
       │   (ndarray, std)     │  │  (tract or candle)   │
       └──────────┬───────────┘  └──────────┬───────────┘
                  │                         │
                  └────────────┬────────────┘
                               ▼
                    ┌──────────────────────┐
                    │     deepbiomat-core      │
                    │ (no_std, zero deps)  │
                    └──────────────────────┘
```

### `deepbiomat-core` — `no_std` foundation

- `Composition`, `Structure`, `Element`, units (with newtypes via `nutype` or hand-rolled).
- Zero non-`core` deps; `alloc` only when needed.
- All types `Send + Sync` where it makes sense.
- Serialization behind `serde` feature flag.

### `deepbiomat-features` — featurization

- Port subset of P2's featurizers selected for inference-time usefulness.
- `Featurizer` trait analogous to matminer's `BaseFeaturizer`.
- Output: `ndarray::Array1<f32>` or `Array2<f32>` for batches.
- Pure functions; no global state.

### `deepbiomat-infer` — model inference

- `Model` trait with `load(path)`, `predict(features) -> Output`.
- Backend abstraction: pick **one** backend at `v0.4.0` (`tract` for ONNX, `candle` for native, or both behind features).
- Models loaded from disk; no embedded weights.
- Optional `parallel` feature using `rayon`.

### `deepbiomat-py` — Python bindings

- One `#[pymodule]` per Rust module, mirroring Rust API.
- Wheels built by maturin for `manylinux`, `macos`, `windows` × `cp310..cp313` (or current latest stable Python).
- Type stubs (`.pyi`) generated and shipped.
- No business logic in this crate — only thin wrappers + type conversions.

### `deepbiomat-cli` — CLI

- `clap`-based, derive macros.
- Subcommands: `featurize`, `predict`, `info`.
- Reads config from `--config`, env, or stdin.
- Output formats: human, JSON, JSON Lines.

## 3. Determinism + Reproducibility

- All RNG-using code accepts a seed.
- Floating-point ops follow IEEE 754 default; no `--ffast-math`.
- Inference outputs reproducible bit-for-bit on the same hardware + Rust toolchain.
- CLI's `predict` command outputs include a `provenance` block: model SHA-256, input SHA-256, library version.

## 4. Quality Gates

| Stage | Tooling |
|---|---|
| Format | `cargo fmt` |
| Lint | `cargo clippy --all-targets --all-features -- -D warnings` |
| Test | `cargo nextest run` |
| Doc test | `cargo test --doc` |
| Coverage | `cargo llvm-cov` (≥ 80% on `crates/deepbiomat-core` and `deepbiomat-features`) |
| Bench | `cargo bench` (criterion) — perf regression detection in CI on labelled PRs |
| Security | `cargo audit` + `cargo deny check` |
| Unused deps | `cargo machete` |
| MSRV | `cargo msrv verify` |
| Binary size | `cargo bloat --release` — fail CI if CLI > budget |
| Python wheels | `maturin build` matrix in CI |
| Type stubs | `pyright` against the generated `.pyi` |

## 5. Dependency Policy

- `deepbiomat-core`: zero non-`core` deps. `serde` only behind feature flag.
- All other crates: each direct dep needs a one-line rationale in `Cargo.toml` comment.
- License allowlist enforced by `cargo deny`: MIT, Apache-2.0, BSD variants, ISC, MPL-2.0 (file-level), Unicode.
- License denylist: GPL family without exception decision recorded in `docs/adr/`.
- Bump policy: patch → auto, minor → review, major → ADR.

## 6. MSRV Policy

- Declared in `rust-toolchain.toml` and each `Cargo.toml` (`rust-version`).
- Bumps require a minor version bump and a `CHANGELOG.md` entry.
- CI tests both MSRV and latest stable.

## 7. Release Engineering

- Conventional Commits drive changelog (via `git-cliff` or similar).
- Tagging `vX.Y.Z` triggers `release.yml`:
  - Publish each crate to crates.io in dependency order.
  - Build wheels for `deepbiomat-py` via maturin → publish to PyPI via Trusted Publishing.
  - Create GitHub Release with attached binaries (CLI for major OSes).
  - Mint Zenodo DOI.
- Pre-releases use `vX.Y.Z-rc.N`.

## 8. Anti-Patterns to Refuse

- **Crate proliferation** — every new crate must justify itself against existing ones.
- **Logic in `deepbiomat-py`** — bindings only. Logic stays in core/features/infer.
- **`unsafe` without justification** — every `unsafe` block has a SAFETY comment explaining invariants.
- **Hidden allocations in hot paths** — features/infer hot paths allocate at the edge, not in the loop.
- **Network in default features** — opt-in only.
- **`anyhow` in library APIs** — libraries return `thiserror`-derived enums; `anyhow` is for binaries only.
- **Vendored model weights in the repo** — distributed separately, fetched by user.
