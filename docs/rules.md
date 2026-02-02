# SentinelWX Rules

These rules are **mandatory** for all contributors and all AI tools working in this repository.  
If existing code appears to conflict with these rules, **the rules win** and the code must be brought into compliance.

---

## Table of contents

1. [Purpose and scope](#1-purpose-and-scope)  
2. [Data provenance and labeling](#2-data-provenance-and-labeling)  
3. [Rust coding standards](#3-rust-coding-standards)  
4. [Tests-first, Clippy, and fuzzing](#4-tests-first-clippy-and-fuzzing)  
5. [Freeze model and change control](#5-freeze-model-and-change-control)  
6. [Tasks, milestones, and scope](#6-tasks-milestones-and-scope)  
7. [AI tools and `.aicode`](#7-ai-tools-and-aicode)  
8. [Human responsibility](#8-human-responsibility)

---

## 1. Purpose and scope

- SentinelWX is a **safety-critical** weather situational awareness and operations tool.
- Primary goals:
  - Correctness and reliability under stress.
  - Clear provenance and labeling of all data.
  - Predictable, reviewable changes.
- All human and AI work must:
  - Align with `docs/architecture.md`, `docs/roadmap.md`, `docs/milestones.md`, and `docs/tasks.md`.
  - Respect the freeze model defined in `docs/freeze.md`.

---

## 2. Data provenance and labeling

Every data product must be clearly classified as:

- **Official** — Products from official providers (e.g., NWS, SPC, NHC, USGS, etc.).
- **Derived** — Diagnostics, composites, or value-adds built from official or other data.

Rules:

- Never present a derived product as an official one.
- Always attribute sources:
  - In code comments, **and**
  - In `docs/architecture.md` or plugin-specific docs.
- Do not add new external data sources without:
  - Documenting them (endpoints, licensing/terms).
  - Ensuring they are compatible with SentinelWX’s licensing and mission.

---

## 3. Rust coding standards

### 3.1 Safety and error handling

- **No panics in production paths**
  - Avoid `unwrap`, `expect`, `panic!`, and similar in non-test code.
  - If a panic is truly unavoidable, it must be:
    - Explicitly documented in code with a clear justification, **and**
    - Mentioned in `docs/architecture.md`.

- **No `unsafe` by default**
  - `unsafe` is forbidden unless:
    - There is no reasonable safe alternative.
    - The block is tightly scoped and heavily commented.
    - The rationale is recorded in code and in `docs/architecture.md`.
  - Any proposed `unsafe` must be called out for explicit human review.

- **Error handling**
  - Prefer explicit error types and `Result` over silently ignoring errors.
  - Log operationally relevant failures with structured logging (e.g., via `tracing`).
  - Do not swallow errors “for convenience”; propagate or handle them meaningfully.

### 3.2 Style and structure

- Follow idiomatic Rust patterns consistent with the rest of the codebase.
- Keep modules cohesive and focused; avoid god-objects or huge files where possible.
- Public APIs should be:
  - Well-named.
  - Documented with `///` doc comments when non-trivial.

---

## 4. Tests-first, Clippy, and fuzzing

Testing and static analysis are **central** to SentinelWX. Code is not considered done until it is clean under tests, linting, and (where applicable) fuzzing.

### 4.1 Tests-first principle

When adding or changing behavior:

- Prefer to write or update tests **before** or alongside the implementation.
- Cover:
  - Happy paths.
  - Edge cases and error paths.
  - Any regressions being fixed (add a regression test).

Types of tests:

- **Unit tests**: for pure logic, parsing, small components.
- **Integration tests**: for HTTP endpoints, background workers, cross-crate flows.
- **Doc tests**: where appropriate, to ensure documentation examples stay correct.

### 4.2 Mandatory commands

Before marking a task as done or opening a PR, the following must hold:

- `cargo test` succeeds (no failing tests).
- `cargo clippy --all-targets --all-features -- -D warnings` succeeds (no warnings).
- Where fuzz targets exist for the touched code:
  - `cargo fuzz run <target>` has been run for a reasonable duration (or under CI) and does not produce crashes or panics.

If any of these fail, you must:

- Use the output to understand the issue.
- Fix the underlying cause (not just silence the symptom).
- Re-run until the tree is clean.

### 4.3 Scope of tests

- Do not add large features without corresponding tests.
- Do not weaken existing tests to “force things green.”
- When changing behavior:
  - Update tests to reflect the *intended* new behavior.
  - Remove or adjust obsolete tests explicitly; do not comment them out and leave them.

---

## 5. Freeze model and change control

SentinelWX uses a **freeze model** to protect stable, critical files and modules.

### 5.1 States

As defined in `docs/freeze.md`, each file or path is in one of three effective states:

- `mutable`
  - Default state.
  - Normal edits allowed, subject to all other rules.

- `review_only`
  - AI tools must not modify these files.
  - Humans may modify them sparingly, with explicit justification in the PR.

- `frozen`
  - No edits allowed except:
    - Critical bug fixes.
    - Security fixes.
  - Such changes must be:
    - Narrow and well-reasoned.
    - Called out explicitly in the PR description.

### 5.2 Implementation

- The authoritative list of frozen/review-only paths is `docs/freeze.md`.
- Files may also contain in-file markers, for example:

  ```rust
  // SENTINELWX-FROZEN: Do not modify except for critical bug/security fixes with human approval.
  // SENTINELWX-REVIEW-ONLY: AI must not modify this file; human edits only with justification.
If docs/freeze.md and in-file markers disagree:

Treat the file as frozen and seek clarification.

5.3 Rules
Do not:

Perform broad refactors that touch frozen or review_only files.

Reformat frozen files.

Move, rename, or delete frozen files without explicit human instruction.

When freezing or unfreezing:

Update docs/freeze.md.

Add, update, or remove the in-file marker.

Ensure the module is well-tested before freezing.

6. Tasks, milestones, and scope
Work must be task-driven, not free-form.

docs/roadmap.md defines high-level milestones.

docs/milestones.md tracks progress and links milestones to tasks.

docs/tasks.md is the source of truth for concrete, actionable work.

Rules:

Do not invent new large features without adding or updating entries in:

docs/roadmap.md (scope-level change),

docs/milestones.md (milestone-level),

docs/tasks.md (fine-grained tasks).

Do not silently change the scope of an existing task.

If scope must change, edit the task description and/or add a new task.

Only mark tasks complete when:

Acceptance criteria are met.

Tests, Clippy, and fuzzing (where relevant) are clean.

Any necessary documentation updates are done.

7. AI tools and .aicode
AI tools (Claude Code, Antigravity, and similar) are allowed but tightly controlled.

7.1 Required reading
Before editing, an AI tool must:

Load .aicode from the repo root (primary AI instructions).

Read and obey:

docs/rules.md (this file),

docs/architecture.md,

docs/roadmap.md,

docs/milestones.md,

docs/tasks.md,

docs/freeze.md.

If anything in the code conflicts with these documents, the documents win.

7.2 Behavior requirements
AI tools must:

Pick work only from docs/tasks.md, unless the human explicitly assigns a different task.

Work on one task at a time.

Before making edits:

Check docs/freeze.md and in-file markers to avoid frozen / review_only files.

For each task:

Read the task description and acceptance criteria.

Identify which files will be touched.

Implement tests and code to satisfy the task, with minimal, reviewable changes.

After edits:

Run cargo test.

Run cargo clippy --all-targets --all-features -- -D warnings.

Run relevant fuzz targets if they exist for the touched code.

Use failure output to correct issues and repeat until clean.

Only when the task is truly complete may the AI:

Change the corresponding checkbox in docs/tasks.md from [ ] to [x].

Suggest (but not enforce) freezing particularly stable modules, deferring the actual freeze decision to a human.

7.3 Prohibited AI behaviors
AI tools must not:

Edit frozen or review_only files unless a human explicitly overrides for a specific change.

Bypass tests, Clippy, or fuzzing to “save time.”

Add new dependencies or introduce new external services without clear justification and documentation.

Modify licensing, headers, or legal text.

Mass-reformat or mass-refactor the codebase.

8. Human responsibility
Even with strong rules and AI assistance, humans are ultimately responsible for:

Reviewing all changes.

Enforcing these rules.

Deciding when to freeze or unfreeze files.

No change should be merged without:

A human reading the diff.

Confirming:

Tests, Clippy, and fuzzing (where applicable) are clean.

The change is within scope and consistent with this document.

If you are unsure whether a change violates these rules, err on the side of caution and discuss it before merging.
