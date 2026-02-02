# Tasks and milestone work items

This file defines the concrete work items for SentinelWX, organized by roadmap milestone.  
All AI tools and human contributors must treat this file as the **single source of truth** for “what to do next,” unless a human explicitly overrides it.

If anything in existing code or other docs conflicts with this file, **`.aicode` and `docs/rules.md` win first**, then this file.

## 1. General rules

These rules apply to **every** task in this file:

- Tasks are organized by milestone (`M0`, `M1`, …) to match `docs/roadmap.md`.
- Each task has:
  - A unique ID (e.g., `M0-001`).
  - A short description.
  - A list of files expected to be touched.
  - Explicit acceptance criteria.
- Tasks must be completed **one at a time**:
  - AI tools must not work on multiple tasks concurrently.
  - Humans should avoid mixing unrelated tasks into a single PR.
- **Tests‑first discipline**:
  - For any behavioral change or new functionality:
    - Write or update tests **before** or alongside code changes.
    - Prefer to start by making tests fail in the intended way, then implement the minimal code to make them pass.
- Required command loop for completion:
  - `cargo test`
  - `cargo clippy --all-targets --all-features -- -D warnings`
  - `cargo fuzz run <target>` (for each relevant fuzz target, when such harnesses exist for the code being changed)
- A task may only be marked `[x]` when:
  - All acceptance criteria for that task are met.
  - All required commands above have been run and pass clean (no failures, no clippy warnings).
  - The task’s changes are consistent with `.aicode`, `docs/rules.md`, `docs/architecture.md`, and `docs/freeze.md`.

## 2. AI obligations for tasks

AI coding tools (Claude Code, Antigravity, etc.) must:

1. **Select tasks from this file.**
   - By default, choose the **first unchecked** (`[ ]`) task in the lowest‑numbered milestone that is not marked complete, unless the user explicitly specifies a task ID.
   - Never invent new tasks or modify IDs on their own.
   - Do not re‑order tasks unless a human requests it.

2. **Work on exactly one task at a time.**
   - Once a task is selected (e.g., `M0-001`), the AI must:
     - Focus all changes on that task’s scope and listed files (plus any clearly necessary supporting files).
     - Avoid “opportunistic” cleanups in other areas unless they are directly required to complete the task.
   - A new task may only be selected after:
     - The current task is fully complete and marked `[x]`, or
     - The user explicitly instructs the AI to pause or abandon it.

3. **Respect freeze states.**
   - Before editing any file:
     - Check `docs/freeze.md` and in‑file `SENTINELWX-FREEZE-STATE` markers.
     - Do not modify files marked `frozen`.
     - Do not modify `review_only` files unless the user explicitly instructs the AI to do so for a specific, narrow change.
   - If a task appears to require editing a `frozen` or `review_only` file:
     - The AI must call this out explicitly and ask the user or maintainer to change the freeze state or perform the edit manually.

4. **Follow the command loop on every task.**
   - For the selected task:
     - Update or add tests.
     - Implement minimal code changes.
     - Run:
       - `cargo test`
       - `cargo clippy --all-targets --all-features -- -D warnings`
       - `cargo fuzz run <target>` for each relevant fuzz target (where harnesses exist).
     - Iterate on code and tests until all commands pass clean.
   - Only after this loop is clean may the task’s checkbox be changed to `[x]`.

5. **Report exactly what changed.**
   - In explanations to the user, the AI must:
     - List the task ID being worked on.
     - List all files touched.
     - Confirm that each file was allowed by freeze rules.
     - Confirm which commands were run and that they passed clean.

## 3. Task format and conventions

Each task below follows this structure:

```markdown
- [ ] <ID>: <Short title>
  - Description: <what this task is about>
  - Files to touch: <path1>, <path2>, ...
  - Acceptance criteria:
    - <criterion 1>
    - <criterion 2>
    - <tests/clippy/fuzz criteria>
Notes:

The “Files to touch” list is indicative, not exhaustive:

Additional files may be modified only if necessary and within the same scope.

When a task is completed:

Change [ ] to [x].

Do not change the ID or wording, unless a human explicitly instructs it.

If a task becomes obsolete or is superseded:

A human should:

Add a note under the task.

Optionally add a new task with a new ID.

AI tools must not delete tasks or change their meaning on their own.

4. Milestone M0 – Bootstrapping and control layer
High‑level goal (see docs/roadmap.md):
Establish a clean, minimal Rust skeleton, core docs, and strict AI control model; no real weather functionality yet.

M0 tasks
 M0-001: Verify and align core docs and freeze registry

Description: Ensure .aicode, docs/rules.md, docs/freeze.md, and docs/README.md are internally consistent and accurately describe the current repo layout and freeze states.

Files to touch: .aicode, docs/rules.md, docs/freeze.md, docs/README.md

Acceptance criteria:

All references between .aicode, docs/rules.md, and docs/freeze.md are consistent (freeze states, AI responsibilities, task workflow).

docs/README.md lists and briefly describes all core docs, including docs/freeze.md, docs/tasks.md, docs/milestones.md, and docs/ai_coding_guidelines.md.

For any file listed as review_only or frozen in docs/freeze.md, the in‑file SENTINELWX-FREEZE-STATE marker matches.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant fuzz targets (if any exist for affected code) are run and pass clean.

 M0-002: Establish initial test harness structure

Description: Create or refine the basic test harness layout for the crate, including top‑level tests/ modules and any minimal smoke tests required by the architecture.

Files to touch: Cargo.toml, src/lib.rs, src/main.rs (if present), tests/ (new or existing files), docs/architecture.md

Acceptance criteria:

There is a clear tests/ directory structure consistent with docs/architecture.md (e.g., unit tests close to modules, integration tests in tests/).

Minimal smoke tests exist that:

Compile the crate.

Exercise basic startup paths without real external dependencies.

Tests are documented briefly in docs/architecture.md or a referenced tests section.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Any fuzz harness scaffolding (if added) compiles; fuzz targets run clean if invoked.

 M0-003: Wire AI coding guidelines to actual repo structure

Description: Ensure docs/ai_coding_guidelines.md accurately explains how an AI agent should read the repo, select tasks from this file, respect freeze rules, and run the required commands.

Files to touch: docs/ai_coding_guidelines.md, .aicode, docs/tasks.md

Acceptance criteria:

docs/ai_coding_guidelines.md:

Explains which files to read first (README.md, .aicode, docs/rules.md, this docs/tasks.md, docs/freeze.md).

Provides a concrete loop: pick a task, write/update tests, implement minimal code, run cargo test, run cargo clippy --all-targets --all-features -- -D warnings, run relevant cargo fuzz run <target>, iterate until clean, then tick the checkbox.

Describes how to behave when encountering frozen or review_only files.

.aicode points to docs/ai_coding_guidelines.md and this docs/tasks.md as control inputs.

This docs/tasks.md file is referenced correctly (path and meaning).

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Any relevant fuzz targets run clean if affected.

 M0-004: Root README alignment with control model

Description: Ensure the root README.md summarizes the project and clearly states the tests + clippy + fuzz philosophy and directs AI users to .aicode and docs/ai_coding_guidelines.md.

Files to touch: README.md, docs/roadmap.md, docs/ai_coding_guidelines.md

Acceptance criteria:

README.md:

Describes SentinelWX as a safety‑critical weather situational awareness/ops tool.

Mentions the tests‑first philosophy and mandatory cargo test, cargo clippy --all-targets --all-features -- -D warnings, and fuzzing (where applicable).

Includes clear links/references to .aicode, docs/ai_coding_guidelines.md, and docs/roadmap.md.

Any references to milestones and tasks in README.md align with docs/roadmap.md and this docs/tasks.md.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant fuzz targets run clean if affected.

 M0-005: Human CONTRIBUTING guide aligned with AI model

Description: Update CONTRIBUTING.md so that human contributors follow the same safety‑critical expectations as AI tools, including freeze, tests, clippy, and fuzz.

Files to touch: CONTRIBUTING.md, docs/freeze.md, docs/ai_coding_guidelines.md

Acceptance criteria:

CONTRIBUTING.md:

Directs contributors to .aicode and docs/ai_coding_guidelines.md if they use AI tools.

Explains the freeze model at a high level and points to docs/freeze.md for details.

Explicitly lists “before PR” checks:

cargo test

cargo clippy --all-targets --all-features -- -D warnings

Relevant cargo fuzz run <target> for code touched by the change, when such harnesses exist.

Language in CONTRIBUTING.md does not contradict .aicode, docs/rules.md, or docs/freeze.md.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant fuzz targets run clean if affected.

5. Milestone M1 – Core runtime skeleton
High‑level goal (see docs/roadmap.md):
Create the minimal runtime skeleton: configuration loading, logging, and a clean main entry point, without connecting to real external weather sources.

M1 tasks
 M1-001: Configuration loading scaffold

Description: Implement a minimal configuration module that can load basic settings (e.g., log level, bind address) from a config file or environment, without external APIs.

Files to touch: src/config.rs (or equivalent), src/main.rs or src/bin/sentinelwx.rs, tests/config_*.rs

Acceptance criteria:

There is a clear configuration module with a small, well‑typed config struct.

Unit tests cover:

Happy‑path loading.

Reasonable error handling behavior.

No real external endpoints or secrets are hardcoded.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

If a fuzz harness is added for config parsing, cargo fuzz run <config_target> runs clean.

 M1-002: Logging scaffolding

Description: Introduce a basic, structured logging setup suitable for safety‑critical use (no panics in normal operation, no secrets in logs).

Files to touch: src/logging.rs (or equivalent), src/main.rs, tests/logging_*.rs

Acceptance criteria:

A logging module initializes a structured logger at startup.

Tests cover at least:

Initialization path (no panic).

Basic log emission in a testable way (e.g., via test logger or capturing output).

Logging decisions align with docs/rules.md security and observability guidance.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Any fuzz targets for logging (if present) run clean.

6. Milestone M2 – Data ingestion scaffolding
High‑level goal (see docs/roadmap.md):
Define internal models and placeholder ingestion paths for official weather/hazard data, without full implementation or external API traffic.

M2 tasks
 M2-001: Internal data model skeleton

Description: Create initial Rust types for core weather/hazard concepts (e.g., alerts, grids) as per docs/architecture.md, without binding to any specific external schema.

Files to touch: src/model/*.rs, tests/model_*.rs, docs/architecture.md

Acceptance criteria:

Well‑typed core data structures exist with clear naming and documentation.

Unit tests validate basic invariants (e.g., required fields, simple constructors).

Models are referenced in docs/architecture.md.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Any fuzz targets for model serialization/deserialization (if added) run clean.

7. Milestones M3+ – Placeholder
Higher milestones (M3–M8) will be expanded with concrete tasks as implementation progresses.
For now, treat this section as a placeholder tied to docs/roadmap.md.

 M3-001: (Placeholder) Define initial ingest pipeline tasks

Description: To be defined once M1 and M2 tasks are stable and passing.

Files to touch: docs/tasks.md

Acceptance criteria:

A human maintainer expands M3 tasks in collaboration with docs/roadmap.md and docs/architecture.md.

AI tools must not fill this in autonomously.

8. How to extend this file (humans only)
Only human maintainers should:

Add new tasks.

Change task descriptions, IDs, or acceptance criteria.

Re‑scope milestones.

Guidelines for maintainers:

When adding a new task:

Choose the appropriate milestone.

Assign a unique ID in sequence (e.g., next M1-00X).

Specify:

Files to touch.

Clear acceptance criteria that include:

Tests written or updated.

cargo test passing.

cargo clippy --all-targets --all-features -- -D warnings passing.

Relevant fuzz targets run clean for affected code, if available.

When splitting a task:

Keep the original ID as a historical anchor.

Add new IDs (e.g., M2-010a, M2-010b), and mark the original as superseded.

When marking a task complete:

Confirm that all criteria and command checks have actually been satisfied.

For AI‑completed tasks, review diffs and freeze states before merging.

AI tools must not perform any of the above autonomously; they may only suggest changes when explicitly asked, and humans must decide what to accept.
