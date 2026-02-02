# AI coding guidelines for SentinelWX

This document explains how AI coding assistants (Claude Code, Antigravity, etc.) must behave when working in the SentinelWX repository.  
It complements `.aicode` and `docs/rules.md` and does **not** replace them.

If there is any conflict:

1. `.aicode` wins first.  
2. Then `docs/rules.md`.  
3. Then `docs/freeze.md`.  
4. Then this file, `docs/tasks.md`, and `docs/milestones.md`.

When in doubt, follow the strictest rule and ask a human for clarification.

---

## 1. What to read first

When an AI tool attaches to this repo, it must load and read (in roughly this order):

1. **Root control and overview**
   - `README.md`
     - High‑level description of SentinelWX as a safety‑critical weather situational awareness/ops tool.
     - Pointers to docs and philosophy (tests, clippy, fuzz, freeze).
   - `.aicode`
     - Primary AI instruction file.
     - Defines priorities, allowed behaviors, and how to pick tasks from `docs/tasks.md`.

2. **Core process and rules**
   - `docs/rules.md`
     - Non‑negotiable rules: safety, security, correctness, tests‑first, code standards.
     - Sections 5.2+ define freezing as a human decision and require respecting `docs/freeze.md`.
   - `docs/freeze.md`
     - Defines `mutable`, `review_only`, and `frozen`.
     - Explains how humans freeze/unfreeze and how AI must treat each state.

3. **Planning and status**
   - `docs/roadmap.md`
     - High‑level milestones M0–M8 and the intended evolution of the system.
   - `docs/milestones.md`
     - Short connector: milestone goals and status (`Not started / In progress / Complete`).
     - Points to `docs/tasks.md` for actual work items.
   - `docs/tasks.md`
     - The **single source of truth** for concrete tasks.
     - Organized by milestone with IDs, files to touch, and acceptance criteria.

4. **Architecture and design**
   - `docs/architecture.md`
     - High‑level architecture, components, data flow, and test layout.

5. **Other helpful docs**
   - `CONTRIBUTING.md`
     - Human contributor expectations; AI should mirror these checks when making changes.
   - `docs/README.md`
     - Index of all docs and their roles.

The AI must not begin editing code or docs until it has at least skimmed all of the above.

---

## 2. Understanding freeze states

The freeze model is defined in detail in `docs/freeze.md`.  
AI tools must actively check and respect freeze states **before** modifying any file.

### 2.1 States

Each relevant file can be:

- `mutable`
- `review_only`
- `frozen`

The canonical information lives in:

- `docs/freeze.md` (registry table).
- In‑file markers, for example:
  - Rust: `// SENTINELWX-FREEZE-STATE: mutable`
  - Markdown: `<!-- SENTINELWX-FREEZE-STATE: review_only -->`
  - Config/TOML: `# SENTINELWX-FREEZE-STATE: frozen`

### 2.2 AI behavior per state

- `mutable`
  - AI may read and modify as needed, **within the scope of the current task**.
  - Changes must still follow all rules: tests‑first, minimal diffs, safety‑critical standards.

- `review_only`
  - AI may read but **must not modify** by default.
  - Exception: If the user explicitly instructs the AI to make a narrow change (e.g., fix a typo) and the file is `review_only` (not `frozen`), the AI may propose and apply the minimal edit.
  - If a task appears to require edits to a `review_only` file:
    - AI must call this out explicitly and ask the user or maintainer to either:
      - Temporarily move the file back to `mutable`, or
      - Perform the change manually.

- `frozen`
  - AI must **never modify** or propose edits.
  - If the user asks for a change to a `frozen` file:
    - AI must explain that the file is frozen and request that a human:
      - Update `docs/freeze.md` and the in‑file marker to unfreeze or downgrade the file, **before** any edits occur.
  - AI must not create “work‑around” code that duplicates frozen logic elsewhere to avoid the freeze.

When there is any discrepancy between markers and the registry, the AI must assume the stricter interpretation and alert the user.

---

## 3. How to pick and run tasks

All AI work must be driven by **explicit tasks** from `docs/tasks.md` or from a direct user instruction that references a specific task.

### 3.1 Selecting a task

Unless the user specifies a task ID:

1. Open `docs/tasks.md`.
2. Identify the lowest‑numbered milestone that is not obviously complete (based on checkboxes and `docs/milestones.md`).
3. Within that milestone, choose the first unchecked (`[ ]`) task.
4. Announce which task you are working on, including its ID (e.g., `M0-003`).

Rules:

- Do **not** invent new tasks or IDs.
- Do not mark tasks as obsolete, superseded, or deleted; only humans may do this.
- Do not work on multiple tasks at once.
- If the user explicitly names a task (e.g., “work on `M1-001`”):
  - Obey the user, but still respect freeze rules and `.aicode`.

### 3.2 One task at a time

For the selected task:

- Scope all changes to:
  - The “Files to touch” listed for that task, plus any clearly necessary dependencies.
- Avoid drive‑by refactors or style changes in unrelated files.
- If you discover issues outside the task’s scope:
  - Describe them briefly.
  - Suggest they be captured as new tasks (by humans) in `docs/tasks.md`.

The AI must not:

- Start a new task before finishing the current one.
- Mark any task `[x]` until the required command loop has been run and all acceptance criteria are satisfied.

---

## 4. Tests‑first and minimal‑change philosophy

SentinelWX is safety‑critical.  
AI tools must assume that correctness, robustness, and traceability matter more than speed or cleverness.

### 4.1 Tests first

Whenever you implement or change behavior:

1. Identify relevant tests:
   - Existing unit tests near the code.
   - Integration tests in `tests/`.
   - Any fuzz targets that touch the area.
2. If tests do not exist:
   - Add minimal, focused tests that express the intended behavior (happy path and key edge cases).
3. Make tests fail in the intended way (if applicable).
4. Implement the **smallest** necessary code change to make tests pass.

Tests must be deterministic and must not depend on real external services or non‑reproducible data.

### 4.2 Minimal diffs

For any task:

- Prefer small, isolated changes.
- Avoid large refactors unless the task explicitly calls for them.
- Do not rename, move, or reorganize code unless:
  - It is required by the task.
  - The change is clearly documented.
- Do not rewrite entire files or APIs without a human explicitly instructing you to do so.

---

## 5. Mandatory command loop

For **every** task, AI tools must run the following commands and use their output to drive fixes before marking the task complete.

### 5.1 Required commands

For the selected task:

1. `cargo test`
   - Run the full test suite unless the user explicitly narrows the scope.
   - If tests fail:
     - Read the failures carefully.
     - Update tests and/or code minimally until all tests pass.

2. `cargo clippy --all-targets --all-features -- -D warnings`
   - Treat all clippy warnings as errors.
   - Fix issues rather than suppressing them, unless suppression is explicitly justified by the rules and architecture.
   - If a warning cannot reasonably be fixed:
     - Explain why in the change notes.
     - Use the narrowest possible `allow` with comments referencing the rationale.

3. `cargo fuzz run <target>`
   - For each fuzz target relevant to the code you changed (if those harnesses exist), run:
     - `cargo fuzz run <target>`
   - If fuzzing finds a crash or failure:
     - Treat it as a bug.
     - Fix the underlying issue.
     - Add or update tests where appropriate, so the bug cannot silently reappear.

These commands must be run **after** implementing the code changes and again after each significant fix, until all are clean.

### 5.2 When commands fail

If any of the required commands fail:

- Do **not** mark the task as complete.
- Use the failure output as guidance:
  - For test failures:
    - Improve code or tests to make behavior correct and deterministic.
  - For clippy failures:
    - Fix lints or justify minimal, explicit exceptions.
  - For fuzz failures:
    - Fix crashes, panics, or unexpected behavior.
- Iterate:
  - Apply minimal changes.
  - Re‑run the relevant commands.
- Only when the full set passes clean may you change `[ ]` to `[x]` for the task.

If you cannot make progress without broad or risky changes, stop and ask the user for guidance.

---

## 6. Presenting changes and diffs

When you finish working on a task (or an iteration of it), you must provide a clear summary of what you did.

### 6.1 Required information

For each completed or proposed change:

- Task:
  - Task ID and short title (e.g., `M0-003: Wire AI coding guidelines to actual repo structure`).
- Files touched:
  - List all modified files, with a quick note per file.
  - For each file, indicate the freeze state **at the time of change** (`mutable`, `review_only`, or `frozen`).
- Summary of changes:
  - A brief, focused description of what changed and why.
  - Highlight any behavior changes or new public interfaces.
- Tests and commands:
  - State explicitly which commands were run:
    - `cargo test`
    - `cargo clippy --all-targets --all-features -- -D warnings`
    - `cargo fuzz run <target>` (list the targets)
  - Confirm whether each command passed clean.
  - Note any remaining warnings or limitations.

### 6.2 Diffs

When your environment supports it (e.g., IDE integration, pull request comments):

- Show diffs grouped by file.
- Keep changes as small and readable as possible.
- Avoid mixing unrelated changes in a single diff.

If a human requests more granular detail (e.g., function‑by‑function explanation), obey the request.

---

## 7. Alignment with human contributors

AI tools must behave as if they are careful, junior contributors working under strict review from human maintainers.

### 7.1 Matching CONTRIBUTING expectations

- Mirror `CONTRIBUTING.md`:
  - Run the same “before PR” checks:
    - `cargo test`
    - `cargo clippy --all-targets --all-features -- -D warnings`
    - Relevant `cargo fuzz run <target>` for touched areas.
  - Respect code review and style expectations.
- Do not bypass review:
  - Treat human maintainers as final decision‑makers.
  - Do not override rules or guidelines on your own.

### 7.2 No scope drift

- Do not:
  - Add new milestones.
  - Change roadmap content.
  - Invent undocumented features.
- Only work on:
  - Explicit tasks from `docs/tasks.md`, and/or
  - Direct user instructions that are consistent with `.aicode` and `docs/rules.md`.

If a user asks for something that conflicts with the rules (e.g., “just ignore clippy,” “edit a frozen file”), you must:

- Explain the conflict clearly.
- Offer safer alternatives.
- Proceed only if the user explicitly acknowledges and directs you, and only where the rules allow such exceptions.

---

## 8. Summary checklist for AI tools

Before starting work:

- [ ] Read `README.md` and `.aicode`.
- [ ] Read `docs/rules.md`, `docs/freeze.md`, `docs/roadmap.md`, `docs/milestones.md`, and `docs/tasks.md`.
- [ ] Skim `docs/architecture.md` for relevant components.

For each task:

- [ ] Select one task from `docs/tasks.md` (or use the user‑specified ID).
- [ ] Check freeze state for all files you intend to touch.
- [ ] Write or update tests **first** (or in lockstep with code).
- [ ] Implement the minimal code changes.
- [ ] Run `cargo test`.
- [ ] Run `cargo clippy --all-targets --all-features -- -D warnings`.
- [ ] Run `cargo fuzz run <target>` for relevant fuzz targets, if they exist.
- [ ] Fix issues and re‑run commands until all pass clean.
- [ ] Summarize changes, list touched files and freeze states, and report command outcomes.
- [ ] Only then, change the task’s checkbox from `[ ]` to `[x]`.

AI tools must treat these guidelines as mandatory operational procedures, not suggestions.
