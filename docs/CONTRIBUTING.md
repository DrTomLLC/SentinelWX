# Contributing to SentinelWX

Thank you for your interest in contributing to SentinelWX.  
This is a safety‑critical project, and contributions must respect strict rules around reliability, data provenance, and security. [web:157][web:170]

## Ground rules

Before opening issues or PRs, you must read:

- `docs/rules.md`
- `docs/architecture.md`
- `docs/roadmap.md`
- `docs/ai_coding_guidelines.md` (if you use AI tooling)

By contributing, you agree to follow these documents and keep them up to date when your changes affect them.

Key expectations:

- No panics in production paths.
- No `unsafe` without prior discussion and explicit justification.
- Exhaustive error handling with meaningful messages and logs.
- Clear separation between official data and derived/diagnostic products.

## Workflow

1. Fork the repository and create a feature branch.
2. Open an issue (or link to an existing one) describing:
   - The problem you are solving.
   - The relevant roadmap milestone in `docs/roadmap.md`.
3. Implement the change in small, reviewable commits.
4. Add or update tests:
   - Unit tests for logic and parsing.
   - Integration tests for HTTP/API behavior where applicable.
5. Update documentation:
   - `docs/architecture.md` if the design or data flows change.
   - `docs/rules.md` if you are proposing new constraints.
   - `docs/roadmap.md` if you are adding or re‑scoping work.
6. Open a pull request referencing the issue and the affected milestone.

## Code style and tooling

- Rust stable toolchain.
- Consistent formatting (`cargo fmt`) and linting (`cargo clippy` where configured).
- Structured logging with `tracing` and no ad‑hoc `println!` debugging in committed code.

If you use AI assistants, you must:

- Ensure all generated code follows `docs/ai_coding_guidelines.md`.
- Manually review every line for safety, correctness, and style.

## Reporting issues

When filing issues:

- Include environment details (OS, Rust version, how you ran SentinelWX).
- Provide minimal reproduction steps and logs where possible.
- Indicate whether this affects a current roadmap milestone.

Security‑sensitive issues should not be filed publicly.  
Use the private contact method described in `docs/rules.md` (to be defined) instead. [web:171]
