# SentinelWX

SentinelWX is a safety‑critical weather situational awareness and operations tool.  
It is designed for incident command, emergency management, and serious weather users who need trustworthy, stress‑resistant information, not “toy” visualizations.

The project is implemented in Rust with a strong emphasis on:

- Predictable, well‑typed behavior.
- No panics or unchecked error paths in normal operation.
- Aggressive linting with `cargo clippy --all-targets --all-features -- -D warnings`.
- Extensive testing and fuzzing where applicable.

If you are contributing code or using an AI coding assistant, you must treat SentinelWX as safety‑critical software.

---

## Project goals (high level)

SentinelWX aims to:

- Ingest official weather and hazard data from trusted sources.
- Normalize that data into an internal, consistent model.
- Expose it through:
  - A map‑centric user interface suitable for use under stress.
  - APIs and hooks for operational workflows (IC, EM, chase ops, etc.).
- Maintain strict safety, security, and reliability guarantees at all times.

The long‑term, milestone‑based plan is documented in `docs/roadmap.md`.

---

## Repository layout

Key paths:

- `src/`
  - Rust crate source code (core runtime, data models, ingest, etc.).
- `tests/`
  - Integration and higher‑level tests.
- `docs/`
  - Architecture, roadmap, rules, milestones, tasks, freeze policy, and AI guidelines.
  - See [Documentation overview](#documentation-overview).

Control files:

- `.aicode`
  - Primary instructions for AI coding assistants (Claude Code, Antigravity, etc.).
- `docs/rules.md`
  - Non‑negotiable rules for safety, coding standards, and process.
- `docs/freeze.md`
  - Freeze model (`mutable`, `review_only`, `frozen`) and central registry.
- `docs/tasks.md`
  - Task list organized by milestone; **all AI work must be driven from here**.
- `docs/milestones.md`
  - Status and short descriptions for milestones M0–M8.

---

## Tests, Clippy, and fuzzing

SentinelWX uses a strict, tests‑first development philosophy:

- Tests:
  - New behavior must come with tests (or improved tests).
  - Tests should be deterministic and not depend on live external services.
- Clippy:
  - `cargo clippy --all-targets --all-features -- -D warnings` is mandatory.
  - All warnings are treated as errors and must be addressed or explicitly justified.[web:35][web:32]
- Fuzzing:
  - Where fuzz harnesses exist (e.g., parsers, model conversions), you must run:
    - `cargo fuzz run <target>`
  - Any crashes or unexpected behavior must be fixed and covered by tests.

**Before any PR or task can be considered complete:**

1. `cargo test` must pass.
2. `cargo clippy --all-targets --all-features -- -D warnings` must pass.
3. All relevant `cargo fuzz run <target>` invocations for touched code must run clean.

This applies equally to human contributors and AI coding assistants.

---

## AI coding assistants

SentinelWX is explicitly designed to be **AI‑drivable**, but under strict control.

If you use an AI coding assistant (Claude Code, Antigravity, etc.):

1. **Read control docs first**
   - `.aicode` – primary AI instructions and priorities.
   - `docs/rules.md` – safety, security, coding, and process rules.
   - `docs/freeze.md` – freeze model and which files are `mutable`, `review_only`, or `frozen`.
   - `docs/tasks.md` – concrete tasks, organized by milestone.
   - `docs/ai_coding_guidelines.md` – detailed operational loop for AI tools.

2. **Task‑driven, one at a time**
   - Pick tasks from `docs/tasks.md` (or use a human‑specified task ID).
   - Work on exactly one task at a time.
   - Do not invent new tasks or change IDs.

3. **Respect freeze states**
   - Never edit files marked `frozen`.
   - Do not edit `review_only` files unless explicitly instructed and only for narrow changes.
   - If a task requires changing a frozen/review_only file, ask a human to adjust the freeze state.

4. **Command loop**
   - For each task:
     - Write/update tests.
     - Implement minimal code changes.
     - Run:
       - `cargo test`
       - `cargo clippy --all-targets --all-features -- -D warnings`
       - `cargo fuzz run <target>` for relevant fuzz targets, when available.
     - Fix issues and re‑run until all pass clean.
     - Only then mark the task checkbox as complete in `docs/tasks.md`.

For detailed behavior, see `docs/ai_coding_guidelines.md`.

---

## Documentation overview

All project documentation lives under `docs/`:

- `docs/architecture.md`
  - High‑level system design, components, and data flows.
- `docs/roadmap.md`
  - Milestone‑based roadmap (M0–M8) describing the evolution of SentinelWX.
- `docs/rules.md`
  - Core rules for safety, coding standards, testing, and AI behavior.
- `docs/freeze.md`
  - Freeze policy and registry for `mutable`, `review_only`, and `frozen` files.
- `docs/tasks.md`
  - Task list keyed to milestones; includes IDs, “files to touch,” and acceptance criteria.
- `docs/milestones.md`
  - Connector between roadmap milestones and their tasks; includes status per milestone.
- `docs/ai_coding_guidelines.md`
  - Detailed, AI‑focused instructions for reading the repo, picking tasks, running commands, and presenting changes.
- `docs/README.md`
  - Index and brief description of all documentation files.

If you are unsure where to start, read:

1. This `README.md`.
2. `.aicode`.
3. `docs/rules.md`.
4. `docs/roadmap.md`.
5. `docs/tasks.md` and `docs/freeze.md`.

---

## Contributing

Human contributors should read:

- `CONTRIBUTING.md` – contribution process, review expectations, and required checks.
- `docs/rules.md` – safety and coding rules.
- `docs/roadmap.md` and `docs/milestones.md` – to understand current priorities.
- `docs/tasks.md` – to see what work is defined and how completion is tracked.

If you use AI tools:

- You are responsible for ensuring the AI follows `.aicode`, `docs/ai_coding_guidelines.md`, `docs/rules.md`, and `docs/freeze.md`.
- Do not merge changes that:
  - Skip tests, clippy, or fuzzing.
  - Modify frozen files without going through the documented freeze/unfreeze process.

Security‑sensitive issues should be reported via the private channel described in `docs/rules.md` (once defined), not through public issue trackers.

---

## License

License information will be added once the project is ready for broader use.  
Until then, treat the code and documentation as “source available for review and collaboration with the maintainer’s explicit consent.”

