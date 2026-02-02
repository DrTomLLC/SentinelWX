# SentinelWX Rules

These rules are non‑negotiable for all contributors and tools.

## 1. Safety, reliability, and scope

- SentinelWX is a safety‑critical and operations‑support tool.
- Prefer correctness and robustness over cleverness or micro‑optimizations.
- Do not expand the project scope without updating `docs/roadmap.md` and `docs/milestones.md`.

## 2. Data provenance

- Clearly distinguish between:
  - Official products (NWS, SPC, NHC, etc.).
  - Derived diagnostics and value‑adds.
- Never mislabel derived products as official.
- Document every new data source and its terms of use in `docs/architecture.md` and relevant plugin docs.

## 3. Rust coding standards

- No panics in production code paths (`unwrap`, `expect`, `panic!`, etc.) unless justified and documented.
- No `unsafe` unless:
  - There is no reasonable safe alternative.
  - The usage is reviewed and explicitly documented in code and `docs/architecture.md`.
- Favor explicit error handling and rich error types.
- Use structured logging (`tracing` or equivalent) for operationally relevant events.

## 4. Tests‑first, linting, and fuzzing

The **default workflow** is:

1. Start from a task in `docs/tasks.md`.
2. Write or update tests to describe the desired behavior:
   - Unit tests for logic, parsing, and small units.
   - Integration tests for HTTP endpoints, background jobs, and cross‑crate flows.
3. Implement the code to make the tests pass. [web:206]

Before considering work “done”:

- `cargo test` must succeed.
- `cargo clippy --all-targets --all-features -- -D warnings` must succeed. [web:208][web:211]
- Where fuzz targets exist for the touched code:
  - Run the relevant `cargo fuzz run <target>` for a reasonable time window.
  - Fix any crashes or panics that appear. [web:209]

If you cannot follow tests‑first strictly (e.g., for a large refactor), you must still:

- Add or update tests before merging to cover the new behavior.
- Ensure tests, clippy, and configured fuzzers are clean.

## 5. Freeze and change control

- File freeze state is controlled by `docs/freeze.md` and in‑file markers.
- Respect the definitions:
  - `mutable` – normal edits allowed.
  - `review_only` – AI must not edit; humans edit sparingly with justification.
  - `frozen` – no edits except critical bug or security fixes with explicit approval.

Rules:

- Do not modify `frozen` or `review_only` files unless explicitly instructed by a human.
- When freezing or unfreezing a file, update `docs/freeze.md` and the in‑file marker.
- Do not perform broad refactors that touch many files without a clear milestone and review plan.

## 6. AI tools

- AI tools (Claude Code, Antigravity, etc.) must:
  - Load `.aicode` from the repo root.
  - Read `docs/rules.md`, `docs/architecture.md`, `docs/roadmap.md`, `docs/milestones.md`, `docs/tasks.md`, and `docs/freeze.md` before editing.
  - Pick tasks only from `docs/tasks.md`.
  - Obey freeze rules and in‑file markers at all times.

- AI tools must:
  - Run tests, clippy, and fuzzers as described above.
  - Use the output to fix issues in a loop until the tree is clean or they hit a clear blocker.
  - Keep changes small and focused on a single task.

If any of these rules conflict with existing code patterns, the rules win.  
Raise an issue or note the inconsistency instead of copying bad patterns.
