# AI Coding Guidelines (SentinelWX)

These rules govern how AI coding assistants (e.g., Claude, Copilot) must be used on this repo. SentinelWX is treated as **safety‑critical** software.

If any generated code conflicts with this document, it **must not** be merged.

## 0. Safety‑critical mindset

- Assume **lives depend on this code**.
- Favor explicit, boring, and defensive code over cleverness.
- Any uncertainty should be resolved by:
  - Writing/expanding tests, and
  - Requesting human review and clarification.

## 1. Hard constraints

These are absolute; never violate them:

1. **No panics in production paths**
   - Do **not** use:
     - `unwrap()`, `expect()`, `panic!()`, `assert!()` (outside of tests), or equivalents.
   - All fallible operations must return `Result` or `Option` and be handled explicitly.
   - If a situation is truly “impossible”, prefer:
     - An explicit error variant, or
     - A documented, carefully considered fallback behavior.

2. **All errors must be handled**
   - Every `Result` and `Option` must be matched and handled.
   - Handle **all** match arms; use `_` only if followed by safe, explicit behavior.
   - No ignored errors (`let _ = some_result;`) unless accompanied by a clear, documented rationale and logging.

3. **Exhaustive logic**
   - For enums and branching logic, handle every case, even if it seems “stupid” or unlikely.
   - Where external inputs are involved (APIs, config), treat them as untrusted:
     - Validate ranges, types, and invariants.
     - Fail **gracefully** with a clear error and logging, never by panicking.

4. **File size limits**
   - Target: **< 250 LOC per file**, hard maximum: **< 400 LOC per file** (excluding comments and whitespace).
   - If a file approaches this limit, split into smaller modules.

5. **Rust edition and toolchain**
   - Code must compile cleanly with:
     - Rust **1.93** and **2024 edition**.
   - Prefer stable features; no nightly‑only APIs.

6. **Extensive testing, in separate files**
   - **Every module and function with logic must have tests.**
   - Tests must live under a separate `tests/` directory and files, not inline with production code:
     - No `#[cfg(test)]` inline unit tests inside source files.
   - Tests must cover:
     - Normal cases.
     - Edge cases.
     - Error conditions and failure paths.
   - When stuck or debugging, **write tests first** or expand existing tests to pin down behavior.

7. **Safety‑critical Rust, always**
   - No `unsafe` blocks, unless there is a documented, reviewed, and justified exception.
   - Favor:
     - Clear ownership and lifetimes.
     - Minimal shared mutable state.
     - Concurrency patterns that avoid data races and deadlocks (e.g., channels, actors, structured concurrency).

8. **No silent degradation without visibility**
   - If you must fall back (e.g., missing data, degraded feed), emit structured logs and propagate a clear status up to callers.
   - The system may continue operating in degraded mode, but operators must be able to see what degraded and why.

## 2. Security constraints

- Do not add:
  - Debug HTTP endpoints that expose internal state without auth.
  - Backdoors or “temporary” shortcuts that bypass checks.
- For any network‑facing code:
  - Validate all input (query params, JSON bodies, headers).
  - Enforce clear separation between read‑only and mutating operations.
- Never:
  - Log secrets or full credential material.
  - Build SQL/command strings by concatenating untrusted input.
  - Introduce dynamic code execution based on untrusted input.
- When exposing the HTTP API:
  - Assume it may eventually be reachable from untrusted networks.
  - Design auth and authorization hooks now, even if initial deployments are local‑only.

## 3. Data and APIs

- Only call APIs documented in `architecture.md` or explicitly referenced in code comments:
  - NWS API, SPC MapServer, NHC GIS, MRMS/QPE, NOHRSC, USGS, NIFC, NCEI/CDO, Ambient Weather, mPING, and vendor APIs where configured. [web:4][web:63][web:7][web:89][web:90][web:95][web:98][web:101][web:102][web:103][web:104][web:107][web:108][web:109][web:116][web:117][web:121][web:133][web:136][web:139]
- Never:
  - Introduce new external services without design approval.
  - Hard‑code secrets or credentials.
  - Assume API responses are well‑formed; validate and handle errors.

## 4. Structure and style

- Respect existing crate/module layout and names from `architecture.md`.
- Keep functions short and single‑purpose; if a function grows large or complex, split it.
- Prefer explicit types over “clever” type inference when clarity is improved.
- Document:
  - Public functions and types with Rustdoc.
  - Non‑obvious invariants and assumptions in comments.

## 5. Logging and observability

- For any non‑trivial error, log with enough context to debug:
  - Source (plugin/module).
  - Operation (e.g., “fetch NWS alerts”, “parse SPC outlook JSON”).
  - Key parameters (e.g., URL, region, time).
- Do not log sensitive data (keys, passwords, full personal info).

## 6. Testing guidance

- For each module:
  - Create a corresponding test file under `tests/` (e.g., `tests/nws_ingest.rs` for `src/plugins/nws/ingest.rs`).
- Tests should:
  - Use recorded or synthetic JSON/GeoJSON/KML samples that represent real API responses.
  - Assert on both success and failure cases.
  - Cover alert rules, schedulers, and any conditional logic thoroughly.

- When behavior is unclear:
  - Write a test that encodes the intended behavior.
  - Adjust implementation to satisfy the tests while respecting the project rules.

## 7. Licensing and provenance

- All generated code must be compatible with:
  - AGPL‑3.0 (open‑source side), and
  - The project’s commercial dual‑licensing model. [web:145][web:149][web:150][web:152][web:156]
- Do not paste or emulate code from incompatible‑license projects.

## 8. Human review

- AI‑generated changes must be reviewed by a human with Rust and domain experience.
- If an AI suggestion conflicts with:
  - This document,
  - `architecture.md`,
  - `rules.md`, or
  - The maintainer’s explicit instructions,
  **it must be discarded or corrected.**
