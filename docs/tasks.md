# Tasks and milestone work items

This file defines the concrete work items for SentinelWX, organized by roadmap milestone.  
All AI tools and human contributors must treat this file as the **single source of truth** for “what to do next,” unless a human explicitly overrides it.

If anything in code or other docs conflicts with this file, **`.aicode` and `docs/rules.md` win first**, then this file.

---

## 1. General rules

These rules apply to **every** task:

- Tasks are organized by milestone (`M0`, `M1`, …) to match `docs/roadmap.md`.
- Each task has:
  - A unique ID (e.g., `M0-001`).
  - A short description.
  - A list of “Files to touch”.
  - Explicit acceptance criteria.
- Tasks must be completed **one at a time**:
  - AI tools must not work on multiple tasks concurrently.
  - Humans should avoid mixing unrelated tasks into a single PR.
- **Tests‑first discipline**:
  - For any behavioral change or new functionality:
    - Write or update tests **before** or in tight lockstep with code changes.
    - Prefer to start by making tests fail in the intended way, then implement the minimal code to make them pass.
- Required tooling loop for completion:
  - Core tools (always, for any code/test change):
    - `cargo fmt`
    - `cargo test`
    - `cargo clippy --all-targets --all-features -- -D warnings`
    - Relevant `cargo fuzz run <target>` for the changed area
    - `cargo llvm-cov` for coverage on touched modules
  - Extended tools (when dependencies, public APIs, or safety‑critical logic are changed), such as:
    - `cargo audit`
    - `cargo deny check`
    - `cargo udeps`
    - `cargo machete`
    - `cargo outdated`
    - `cargo minimal-versions`
    - `cargo msrv`
    - `cargo vet`
    - `cargo supply-chain`
    - `cargo geiger`
    - `cargo semver-checks`
    - SBOM/license tools (`cargo about`, `cargo sbom`, `cargo cyclonedx`)
    - Mutation/safety tools (`cargo mutants`, `cargo careful`) when required by a task or milestone
- **Do not stop at the first failure**:
  - Even if one tool fails (e.g., a test or `cargo audit`), you must continue running the rest of the relevant tools for that task to gather a full picture of all issues.
  - Only after you have all tool outputs do you begin fixing issues and adding tests until **all** required tools are clean or exceptions are explicitly accepted by a human.

A task may only be marked `[x]` when:

- All acceptance criteria for that task are met.
- All required tools (core and, when applicable, extended) have been run and pass clean, or exceptions are explicitly accepted by maintainers.
- The changes are consistent with `.aicode`, `docs/rules.md`, `docs/architecture.md`, and `docs/freeze.md`.

---

## 2. AI obligations for tasks

AI coding tools (Claude Code, Antigravity, etc.) must:

1. **Select tasks from this file.**
   - By default, choose the **first unchecked** (`[ ]`) task in the lowest‑numbered milestone that is not marked complete in `docs/milestones.md`, unless the user specifies a task ID.
   - Never invent new tasks or modify task IDs on their own.
   - Do not reorder tasks unless a human requests it.

2. **Work on exactly one task at a time.**
   - Once a task is selected (e.g., `M0-001`), the AI must:
     - Focus all changes on that task’s scope and listed files (plus any clearly necessary supporting files).
     - Avoid “opportunistic” cleanups in other areas unless they are strictly required to complete the task.
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

4. **Run the full tool chain per task.**
   - For the selected task:
     - Update or add tests.
     - Implement minimal code changes.
     - Run all **core** tools.
     - If dependencies, public APIs, or safety‑critical modules are touched, also run the relevant **extended** tools.
     - Even if one command fails, continue running the rest of the relevant tools to collect all diagnostics.
     - Use all tool outputs to plan fixes and add tests, then re‑run tools until they are clean (or exceptions are explicitly accepted).

5. **Report exactly what changed.**
   - In explanations to the user, the AI must:
     - List the task ID being worked on.
     - List all files touched and their freeze states at the time of change.
     - Confirm which tools were run and summarize their outputs (pass/fail).
     - Confirm that all acceptance criteria and checks are satisfied before marking `[x]`.

---

## 3. Task format and conventions

Each task below follows this structure:

``markdown
- [ ] <ID>: <Short title>
  - Description: <what this task is about>
  - Files to touch: <path1>, <path2>, ...
  - Acceptance criteria:
    - <criterion 1>
    - <criterion 2>
    - Core tools run and clean (fmt/test/clippy/fuzz/llvm-cov).
    - Extended tools run and clean when dependencies/public APIs/safety-critical code are touched.

Notes:

The “Files to touch” list is indicative, not exhaustive:

Additional files may be modified if necessary and within the same scope.

When a task is completed:

Change [ ] to [x].

Do not change the ID or description unless a human explicitly instructs it.

If a task becomes obsolete or is superseded:

A human should:

Add a note under the task.

Optionally add a new task with a new ID.

AI tools must not delete tasks or change their meaning on their own.

4. Milestone M0 – Bootstrapping and control layer
High‑level goal (see docs/roadmap.md):
Establish a minimal Rust skeleton, core docs, and strict AI control/freeze model with tests‑first behavior; no real weather functionality yet.

M0 tasks
 M0-001: Verify and align core docs and freeze registry

Description: Ensure .aicode, docs/rules.md, docs/freeze.md, and docs/README.md are internally consistent and accurately describe the current repo layout, freeze states, and extended tooling model.

Files to touch: .aicode, docs/rules.md, docs/freeze.md, docs/README.md

Acceptance criteria:

.aicode, docs/rules.md, and docs/freeze.md all:

Describe the freeze model consistently.

Reference the same set of core and extended tools (fmt/test/clippy/fuzz/llvm‑cov + audit/deny/udeps/outdated/etc.).

docs/README.md:

Lists and briefly describes all core docs, including docs/freeze.md, docs/tasks.md, docs/milestones.md, and docs/ai_coding_guidelines.md.

Points to docs/tools sections (where present) that describe the toolchain.

For any file listed as review_only or frozen in docs/freeze.md, the in‑file SENTINELWX-FREEZE-STATE marker matches.

Core tools:

cargo fmt run and clean.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> (if any) for code touched by this task run clean.

cargo llvm-cov run without regressions on any touched safety‑critical modules.

Extended tools (if this task touches deps or public APIs; otherwise optional but recommended) run and any failures are either fixed or explicitly documented.

 M0-002: Establish initial test harness structure

Description: Create or refine the basic test harness layout for the crate, including top‑level tests/ modules and minimal smoke tests required by the architecture.

Files to touch: Cargo.toml, src/lib.rs, src/main.rs (if present), tests/ (new or existing files), docs/architecture.md

Acceptance criteria:

There is a clear tests/ directory structure consistent with docs/architecture.md.

Minimal smoke tests exist that:

Compile the crate.

Exercise basic startup paths without real external dependencies.

Tests and their structure are briefly described in docs/architecture.md or a referenced tests section.

Core tools:

cargo fmt run and clean.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> (if any test harness touches fuzzable code) run clean.

cargo llvm-cov shows meaningful coverage for the initial harness and no regressions on new modules.

If this task adds or adjusts dependencies (e.g., for tests), extended tools (cargo audit, cargo deny, cargo udeps, cargo outdated) are run and either clean or any issues are clearly documented.

 M0-003: Wire AI coding guidelines to actual repo structure

Description: Ensure docs/ai_coding_guidelines.md accurately explains how an AI agent should read the repo, select tasks, respect freeze rules, and run the full toolchain (including extended tools and “do not stop at first failure” behavior).

Files to touch: docs/ai_coding_guidelines.md, .aicode, docs/tasks.md

Acceptance criteria:

docs/ai_coding_guidelines.md:

Lists the required docs to read (README.md, .aicode, docs/rules.md, docs/freeze.md, docs/roadmap.md, docs/milestones.md, docs/tasks.md, docs/architecture.md, docs/README.md).

Defines the core tools loop (fmt/test/clippy/fuzz/llvm‑cov) and extended tools for deps/APIs/safety‑critical code.

States explicitly that tools must all be run even if some fail, to gather all diagnostics, before iterating on fixes.

Describes how to behave with mutable, review_only, and frozen files.

.aicode:

Points to docs/ai_coding_guidelines.md and this docs/tasks.md as control inputs for AI behavior.

docs/tasks.md:

References the extended tools model and aligns with the guidelines in this section.

Core tools:

cargo fmt run and clean.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> run clean.

cargo llvm-cov shows no regression on modules touched.

If dependencies or public APIs are touched, extended tools are run and any issues are resolved or explicitly documented.

 M0-004: Root README alignment with control and tooling model

Description: Ensure the root README.md summarizes the project and clearly states the tests + clippy + fuzz + extended tooling philosophy and directs AI users to .aicode and docs/ai_coding_guidelines.md.

Files to touch: README.md, docs/roadmap.md, docs/ai_coding_guidelines.md

Acceptance criteria:

README.md:

Describes SentinelWX as a safety‑critical weather situational awareness/ops tool.

Mentions tests‑first philosophy and mandatory:

cargo fmt

cargo test

cargo clippy --all-targets --all-features -- -D warnings

Fuzzing where applicable

Coverage (cargo llvm-cov) and extended tools for dependency/security work.

Includes clear references to .aicode, docs/ai_coding_guidelines.md, docs/roadmap.md, and docs/rules.md.

Any references to milestones and tasks in README.md align with docs/roadmap.md, docs/milestones.md, and this docs/tasks.md.

Core tools:

cargo fmt run and clean.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> run clean.

cargo llvm-cov shows no regressions for modules touched by this task.

Extended tools are run if dependencies/public APIs are touched, with issues resolved or documented.

 M0-005: Human CONTRIBUTING guide aligned with AI + extended tooling model

Description: Update CONTRIBUTING.md so that human contributors follow the same safety‑critical expectations and extended toolchain as AI tools, including freeze, tests, clippy, fuzz, coverage, and dependency/security tools.

Files to touch: CONTRIBUTING.md, docs/freeze.md, docs/ai_coding_guidelines.md, docs/rules.md

Acceptance criteria:

CONTRIBUTING.md:

Directs contributors who use AI tools to .aicode and docs/ai_coding_guidelines.md.

Summarizes the freeze model and points to docs/freeze.md.

Lists “before PR” checks, including:

Always: cargo fmt, cargo test, cargo clippy --all-targets --all-features -- -D warnings, relevant cargo fuzz run <target>, cargo llvm-cov.

When dependencies or public APIs are changed: cargo audit, cargo deny check, cargo udeps, cargo outdated, and other extended tools as relevant (cargo geiger, cargo semver-checks, etc.).

States that contributors should run all relevant tools even if some fail, and then fix issues until everything is clean.

Language in CONTRIBUTING.md does not contradict .aicode, docs/rules.md, or docs/freeze.md.

Core tools:

cargo fmt run and clean.

cargo test passes.

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> run clean.

cargo llvm-cov shows no regressions in touched modules.

Extended tools run clean when this task touches dependencies or public APIs.

5. Milestone M1 – Core runtime skeleton
High‑level goal (see docs/roadmap.md):
Create the minimal runtime skeleton: configuration loading, logging, and a clean main entry point, without connecting to real external weather sources.

M1 tasks
 M1-001: Configuration loading scaffold

Description: Implement a minimal configuration module that can load basic settings (e.g., log level, bind address) from a config file or environment, without external APIs.

Files to touch: src/config.rs (or equivalent), src/main.rs or src/bin/sentinelwx.rs, tests/config_*.rs, Cargo.toml (if new config crates are added)

Acceptance criteria:

A configuration module exists with a small, well‑typed config struct.

Unit tests cover:

Happy‑path loading.

Key error cases (missing values, invalid types, etc.).

No real external endpoints or secrets are hardcoded.

Core tools:

cargo fmt run and clean.

cargo test passes (including new config tests).

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> for config parsing (if any fuzz harness exists) run clean.

cargo llvm-cov shows meaningful coverage for config logic and no regressions.

If configuration introduces or changes dependencies:

Extended tools run and are clean or any issues explicitly documented:

cargo audit

cargo deny check

cargo udeps

cargo outdated

cargo minimal-versions / cargo msrv as appropriate.

 M1-002: Logging scaffolding

Description: Introduce a basic, structured logging setup suitable for safety‑critical use (no panics in normal operation, no secrets in logs).

Files to touch: src/logging.rs (or equivalent), src/main.rs, tests/logging_*.rs, Cargo.toml (if logging crates are added)

Acceptance criteria:

A logging module initializes a structured logger at startup.

Tests cover:

Logging initialization without panicking.

Emission of basic log events in a testable way (e.g., via a test logger).

Logging behavior aligns with docs/rules.md (no secrets, clear messages).

Core tools:

cargo fmt run and clean.

cargo test passes (including logging tests).

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> (if any logging‑related fuzz harness exists) run clean.

cargo llvm-cov shows coverage for logging paths.

If logging introduces or changes dependencies:

Extended tools run and are clean or issues explicitly documented:

cargo audit

cargo deny check

cargo udeps

cargo outdated

cargo minimal-versions / cargo msrv as needed.

6. Milestone M2 – Data ingestion scaffolding
High‑level goal (see docs/roadmap.md):
Define internal models and placeholder ingestion paths for official weather/hazard data, without full implementation or external API traffic.

M2 tasks
 M2-001: Internal data model skeleton

Description: Create initial Rust types for core weather/hazard concepts (e.g., alerts, grids) as per docs/architecture.md, without binding to any specific external schema.

Files to touch: src/model/*.rs, tests/model_*.rs, docs/architecture.md

Acceptance criteria:

Well‑typed core data structures exist with clear naming and documentation.

Unit tests validate basic invariants (e.g., required fields, constructors, simple validation).

Models are referenced in docs/architecture.md.

Core tools:

cargo fmt run and clean.

cargo test passes (including new model tests).

cargo clippy --all-targets --all-features -- -D warnings passes.

Relevant cargo fuzz run <target> (e.g., for serialization/deserialization) run clean.

cargo llvm-cov shows coverage for model code and no regressions.

If new dependencies are added (e.g., for serialization formats), extended tools run and are clean or issues explicitly documented.

7. Milestones M3+ – Placeholder
Higher milestones (M3–M8) will be expanded with concrete tasks as implementation progresses.
For now, treat this section as a placeholder tied to docs/roadmap.md.

 M3-001: (Placeholder) Define initial ingest pipeline tasks

Description: To be defined once M1 and M2 tasks are stable and passing.

Files to touch: docs/tasks.md

Acceptance criteria:

A human maintainer expands M3 tasks in collaboration with docs/roadmap.md and docs/architecture.md.

AI tools must not fill this in autonomously.

Future tasks under M3–M8 should:

Follow the same format as above.

Explicitly call out when extended tools (e.g., cargo audit, cargo deny, cargo geiger, cargo mutants, cargo semver-checks) are required for acceptance.

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

Core tools run and clean:

cargo fmt

cargo test

cargo clippy --all-targets --all-features -- -D warnings

Relevant cargo fuzz run <target>

cargo llvm-cov for modules touched.

Extended tools run and clean when dependencies/public APIs/safety‑critical modules are affected (e.g., cargo audit, cargo deny, cargo udeps, cargo geiger, cargo semver-checks).

When splitting a task:

Keep the original ID as a historical anchor (with a note, e.g., “superseded by …”).

Add new IDs (e.g., M2-010a, M2-010b) and make the relationships clear.

When marking a task complete:

Confirm that all criteria and tool checks have actually been satisfied.

For AI‑completed tasks, review diffs, freeze states, and tool outputs before merging.

AI tools must not perform any of the above autonomously; they may only suggest changes when explicitly asked, and humans decide what to apply.
