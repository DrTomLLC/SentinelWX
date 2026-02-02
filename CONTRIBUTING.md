# Contributing to SentinelWX

SentinelWX is treated as **safety‑critical** software. Contributions are welcome, but they must follow strict rules.

## Before you start

1. Read:
   - `rules.md`
   - `architecture.md`
   - `ai_coding_guidelines.md`
2. Make sure you understand:
   - No panics (`unwrap`, `expect`, `panic!`, `assert!` in production).
   - No `unsafe` (except in very rare, heavily reviewed cases).
   - All errors must be handled explicitly.
   - All logic must have tests in `tests/`.

## How to contribute

1. **Discuss first (recommended)**
   - Open an issue describing:
     - What you want to change or add.
     - Why it’s needed (ops use case).
   - This is especially important for:
     - New data sources.
     - New plugins.
     - Changes to the core or security model.

2. **Follow the architecture**
   - Add new functionality via:
     - Plugins (`sentinelwx-plugins-*`) when it’s a new data source or integration.
     - `sentinelwx-core` / `sentinelwx-api` when it’s core behavior.
   - Keep files under ~250 LOC (400 max) by splitting modules as needed.

3. **Tests are mandatory**
   - Add or update tests in `tests/` for every new behavior or bug fix.
   - Cover:
     - Normal, edge, and error cases.
   - PRs without adequate tests will not be merged.

4. **Security and privacy**
   - Do not:
     - Add new external endpoints without discussion.
     - Log secrets or sensitive personal information.
     - Introduce any dynamic code execution from untrusted input.
   - Assume SentinelWX may be deployed on exposed networks; design defensively.

5. **Style and tooling**
   - Use Rust 1.93, edition 2024.
   - Run:
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`
   - Before opening a PR, ensure all tests pass.

## Submitting a pull request

1. Fork the repo and create a feature branch.
2. Make your changes with tests.
3. Update relevant docs (`architecture.md`, `roadmap.md`, `rules.md`) if design changes.
4. Open a PR describing:
   - What changed.
   - Why.
   - How it was tested.
5. Be prepared for review focusing on:
   - Safety, error handling, and security implications.
   - Test coverage and clarity.
   - Alignment with project architecture and rules.

Thank you for helping make SentinelWX a trustworthy, professional‑grade tool.
