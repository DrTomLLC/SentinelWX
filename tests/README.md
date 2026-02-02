# SentinelWX Test Layout

All tests live in this `tests/` directory. No tests are allowed inside `src/` files.

## Goals

- Full coverage of all logic:
  - Success paths
  - Edge cases
  - Error and failure behavior
- Clear mapping between `src/` modules and test files.
- Safety‑critical: tests are the primary tool for specifying and verifying behavior.

## Conventions

- For each Rust module or feature area in `src/`, create at least one corresponding test file:

  - `src/core/*.rs` → `tests/core_*.rs`
  - `src/plugins/nws/*.rs` → `tests/plugins_nws_*.rs`
  - `src/plugins/spc/*.rs` → `tests/plugins_spc_*.rs`
  - etc.

- Keep each test file focused:
  - Prefer multiple small test files (`*_ingest.rs`, `*_parse.rs`, `*_rules.rs`) over one huge file.

## What to test

For every non‑trivial function or method:

- Normal cases:
  - Expected inputs → expected outputs.
- Edge cases:
  - Boundary values, strange but valid inputs, empty collections, max/min ranges.
- Error handling:
  - Invalid input formats, missing fields, upstream API errors, timeouts, etc.
  - Ensure the function returns the correct error type and does not panic.

For ingest/parsing:

- Use recorded or synthetic JSON/GeoJSON/KML representative of real upstream responses.
- Include tests for:
  - Well‑formed responses.
  - Malformed or partial responses.
  - Schema changes (extra fields, missing optional fields).

For alert rules:

- Test each rule type with:
  - Conditions not met.
  - Conditions exactly met.
  - Conditions exceeded.
- Verify:
  - When alerts fire (false → true).
  - Cooldowns / deduplication behavior.

## Guidelines

- Do not use `unwrap()` or `expect()` in tests where failure would hide a bug:
  - Prefer explicit matching and `assert!(result.is_err())` with detailed checks.
- Use descriptive test names:
  - `test_nws_alert_parsing_handles_missing_optional_fields`
  - `test_alert_rule_triggers_when_qpe_exceeds_threshold`
- When a bug is found:
  - First write a failing test reproducing it.
  - Then fix the implementation to make the test pass.
