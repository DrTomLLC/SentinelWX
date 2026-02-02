# SentinelWX Tasks

This file contains fine‑grained, checkbox‑driven tasks for humans and AI.  
AI tools must select tasks from here and work on one at a time.

Conventions:

- Each task starts with `- [ ]` (unchecked) or `- [x]` (completed).
- Each task names:
  - Milestone
  - Files to touch
  - Brief acceptance criteria
- Do not add new tasks without human approval.

---

## Milestone 0 – Repo, docs, skeleton

- [ ] M0‑001: Create Rust workspace and main crate  
  Files: `Cargo.toml`, `sentinelwx/Cargo.toml`, `sentinelwx/src/main.rs`  
  Acceptance:
  - `cargo build` succeeds.
  - `main` logs a startup message and exits with code 0.

- [ ] M0‑002: Implement basic `/health` endpoint  
  Files: `sentinelwx/src/main.rs` (or `src/http/health.rs`), tests in `tests/health.rs`  
  Acceptance:
  - `GET /health` returns `200 OK` and a simple JSON or text payload.
  - Integration test in `tests/health.rs` passes.
  - `cargo test` and `cargo clippy --all-targets --all-features -- -D warnings` are clean. [web:165][web:168][web:211]

(Add more M0 tasks as needed.)

---

## Milestone 1 – Core runtime and HTTP API skeleton

- [ ] M1‑001: Config loading and validation  
  Files: `sentinelwx/src/config.rs`, `config/default.toml`, tests in `tests/config.rs`  
  Acceptance:
  - Config loads from file and environment overrides.
  - Invalid config causes startup failure with clear error.
  - Unit and integration tests cover valid/invalid cases.

- [ ] M1‑002: Structured logging setup  
  ...

---

(Continue similarly for Milestones 2–8, breaking down your roadmap entries into 1–5 line tasks.)

## Notes

- AI tools:
  - Must pick the first suitable unchecked task unless the user specifies a different one.
  - Must not mark tasks as complete until all acceptance criteria are met and the tree is clean.
- Humans:
  - May add, re‑scope, or remove tasks as needed.
  - Should keep this file in sync with `docs/milestones.md` and `docs/roadmap.md`.

