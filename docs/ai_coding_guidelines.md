# AI coding guidelines for SentinelWX

This document defines how AI coding assistants (Claude Code, Antigravity, etc.) must behave when working in the SentinelWX repository.  
It **complements** `.aicode` and `docs/rules.md` and does **not** replace them.

If there is any conflict:

1. `.aicode` wins first.  
2. Then `docs/rules.md`.  
3. Then `docs/freeze.md`.  
4. Then this file, `docs/tasks.md`, and `docs/milestones.md`.

When in doubt, follow the **strictest** rule and ask a human for clarification.

---

## 1. What to read first (boot sequence)

When an AI tool attaches to this repo, it must load and read—**in roughly this order**:

1. **Root overview and control**
   - `README.md`  
     High‑level description of SentinelWX, tests+Clippy+fuzz philosophy, and pointers to docs.
   - `.aicode`  
     Primary AI instructions: priorities, allowed behaviors, freeze obedience, and how to pick tasks.

2. **Rules and freeze policy**
   - `docs/rules.md`  
     Non‑negotiable rules: safety, coding standards, tests‑first, tooling (core + extended).
   - `docs/freeze.md`  
     Freeze model (`mutable`, `review_only`, `frozen`), registry, in‑file markers, and behaviors.

3. **Planning and task source**
   - `docs/roadmap.md`  
     Milestones M0–M8 and project evolution.
   - `docs/milestones.md`  
     Milestone goals and status; pointer to tasks.
   - `docs/tasks.md`  
     **Single source of truth** for concrete tasks (IDs, files to touch, acceptance criteria).

4. **Architecture and structure**
   - `docs/architecture.md`  
     Components, data flow, module layout, and test strategy.

5. **Documentation index and contributor info**
   - `docs/README.md`  
     Index of all docs and their roles.
   - `CONTRIBUTING.md`  
     Human expectations; AI must adhere to the same checks and standards.

Do **not** start editing code or docs until you have at least skimmed all of the above.

---

## 2. Freeze states and how to behave

The authoritative freeze model is in `docs/freeze.md`.  
AI must check and respect freeze states **before** modifying any file.

### 2.1 States and sources

Each relevant file uses one of:

- `mutable`
- `review_only`
- `frozen`

The state comes from:

- The registry table in `docs/freeze.md`.
- An in‑file `SENTINELWX-FREEZE-STATE` marker near the top, e.g.:

  - Rust:

    ```rust
    // SENTINELWX-FREEZE-STATE: mutable
    ```

  - Markdown:

    ```markdown
    <!-- SENTINELWX-FREEZE-STATE: review_only -->
    ```

  - TOML/config:

    ```toml
    # SENTINELWX-FREEZE-STATE: frozen
    ```

When registry and marker disagree, assume the **stricter** interpretation and call this out to the user.

### 2.2 Allowed actions per state

- `mutable`
  - You may read and modify the file as needed **within the scope of the current task**.
  - All changes must still follow tests‑first and minimal‑change rules.

- `review_only`
  - You may read but **must not modify** by default.
  - You may only edit if:
    - The user explicitly instructs you to change that file, and
    - The change is narrow and specific (e.g., fix a typo, update a version string).
  - If a task appears to require modifying a `review_only` file:
    - State this explicitly.
    - Ask the user/maintainer to:
      - Temporarily move it to `mutable`, or
      - Edit it themselves.

- `frozen`
  - You must **never modify** or propose edits.
  - If the user asks for changes:
    - Explain that the file is frozen.
    - Ask them to unfreeze/downgrade it (by updating `docs/freeze.md` and the in‑file marker) **before** any edits.
  - Do not implement work‑arounds that replicate the same logic elsewhere just to bypass the freeze.

---

## 3. Task selection and scope

All AI work must be driven by explicit tasks in `docs/tasks.md` or by a user‑given task ID.  
You must **never** invent tasks or change task IDs.

### 3.1 How to pick a task (if the user doesn’t specify)

1. Open `docs/tasks.md`.
2. Identify the lowest‑numbered milestone whose status in `docs/milestones.md` is not `Complete`.
3. Within that milestone, find the first unchecked task (`- [ ]`).
4. Declare that task as your focus, including its ID and title.

If the user specifies a task (e.g., “work on `M1-001`”), use that task instead.

### 3.2 One task at a time

For the selected task:

- Focus all changes on:
  - The listed “Files to touch,” and
  - Any additional files strictly necessary to complete that task.
- Do **not**:
  - Start a second task before finishing or explicitly pausing the current one.
  - Perform drive‑by refactors or style cleanups in unrelated files.
- If you discover unrelated issues:
  - Describe them briefly.
  - Suggest that a human add new tasks to `docs/tasks.md`.

You may stop or switch tasks only if the user tells you to.

---

## 4. Tests‑first, minimal‑change philosophy

SentinelWX is safety‑critical.  
AI must treat **tests, coverage, and analysis tools** as part of the implementation, not an afterthought.

### 4.1 Tests first

For any behavioral change or new feature:

1. Identify relevant tests:
   - Unit tests near the modules you’ll touch.
   - Integration tests under `tests/`.
   - Fuzz targets that exercise the affected code (if they exist).

2. If tests are missing or insufficient:
   - Add minimal, focused tests for:
     - The intended “happy path.”
     - Key edge/error cases.
   - Prefer to make tests fail first, then implement code to make them pass.

3. Keep tests deterministic:
   - No real network calls.
   - No dependence on non‑reproducible timing or external state.

### 4.2 Minimal diffs

- Prefer small, localized changes.
- Avoid large refactors unless the task explicitly calls for them.
- Do not rename or reorganize modules and APIs without a clear reason tied to the task.
- Never rewrite an entire file or subsystem unless the user explicitly instructs you to do so.

---

## 5. Tool chain and mandatory command loop

Your core loop is:

> **Write/update tests → change code → run all relevant tools → fix everything they report (with new tests as needed) → only then mark the task complete.**

### 5.1 Core tools (always run)

For any change that touches code/tests, you must run all of these:

1. `cargo fmt`
   - Ensure code is formatted before presenting or committing diffs.

2. `cargo test`
   - Run the test suite (full by default).
   - If the user narrows the scope (e.g., specific package or test), obey but call that out.

3. `cargo clippy --all-targets --all-features -- -D warnings`
   - Treat all warnings as errors.
   - Prefer to fix lints rather than silencing them.
   - Only use `allow` pragmas when strictly justified and with comments.

4. `cargo fuzz run <target>`
   - For each fuzz target that exercises code you changed:
     - Run `cargo fuzz run <target>`.
   - Fix all fuzz‑found crashes or unexpected behavior, and add tests where appropriate.

5. `cargo llvm-cov`
   - Run `cargo llvm-cov` (or equivalent) to inspect coverage for touched modules.
   - Do not regress coverage for safety‑critical paths.

6. `cargo nextest run` (optional acceleration)
   - You may run `cargo nextest run` as a faster test harness, but it does **not** replace `cargo test` in expectations.

### 5.2 Extended tools (run when deps/APIs/critical code change)

When changes affect dependencies, `Cargo.toml`/`Cargo.lock`, public APIs, or critical logic, you must also run:

**Dependency and supply‑chain:**

- `cargo audit`
  - Use `cargo audit -D` / `--deny-warnings`.
  - Treat advisories and yanked crates as failures until fixed or explicitly waived.

- `cargo deny check`
  - Enforce license and source policy.
  - Any violation is a failure unless explicitly accepted by maintainers.

- `cargo udeps`
  - Remove unused dependencies or document why they remain.

- `cargo machete`
  - Remove or tighten unused features and feature flags.

- `cargo outdated`
  - Review outdated dependencies.
  - For upgrade tasks: update selected crates and ensure everything still passes.

- `cargo minimal-versions` and `cargo msrv`
  - Ensure minimal and MSRV builds still work when touching dependencies or language features.

- `cargo vet` and `cargo supply-chain`
  - For new or sensitive dependencies:
    - Run these to inspect trust and vetting.
    - Do not ignore concerning results; call them out for human decision.

- `cargo about`, `cargo sbom`, `cargo cyclonedx`
  - For release or compliance tasks, generate SBOMs and license reports.

**Safety and API stability:**

- `cargo geiger`
  - Confirm no unintended new `unsafe` usage.
  - Never introduce new `unsafe` blocks unless explicitly instructed and justified.

- `cargo semver-checks`
  - For any change affecting public APIs, ensure semantic versioning contracts are not accidentally broken.

- `cargo careful`
  - When requested or appropriate for critical code, run instrumented builds/tests to catch UB‑like behavior.

- `cargo mutants`
  - For mutation‑testing tasks, run `cargo mutants` and treat surviving mutants as prompts for new tests or explicit justifications.

### 5.3 Do not stop at the first failure

Critical rule:

- **Never stop the loop after the first failing tool.**
- Even if one tool fails (e.g., `cargo test` or `cargo audit`), you must:
  - Continue running the **rest** of the relevant tools in the sequence.
  - Gather **all** diagnostics and outputs in one pass.
- Only after you have a complete picture:
  - Start fixing issues, informed by all tool outputs.
  - Re‑run tools as needed until everything is clean.

Rationale: this avoids a “fail fast, learn nothing else” pattern; the goal is to surface and fix all known problems, not just the first one.

### 5.4 Fixing issues and iterating

When tools report problems:

1. Do **not** mark the task complete.
2. Aggregate all failures:
   - Test failures.
   - Clippy lints.
   - Fuzz crashes.
   - Coverage regressions.
   - Audit/deny/vet/udeps/semver issues, etc.
3. Plan minimal fixes:
   - Adjust code, tests, or configuration in the smallest way that resolves the issue safely.
   - Add new tests where problems indicate missing coverage.
4. Re‑run the relevant subset of tools until:
   - All failures are resolved, or
   - A human explicitly accepts a documented exception.

Only then may you move on to the final step: ticking the task’s checkbox.

---

## 6. Marking tasks complete

A task in `docs/tasks.md` may only be changed from `- [ ]` to `- [x]` when:

1. All acceptance criteria for that task are satisfied.
2. All **required** tools have been run and are clean:
   - Core tools for any code/test change.
   - Extended tools when dependencies, public APIs, or critical logic are touched.
3. All freeze rules are obeyed:
   - No `frozen` file was modified.
   - Any `review_only` edits were explicitly requested and narrowly scoped.
4. You have:
   - Summarized the change.
   - Listed files touched and their freeze states.
   - Reported which tools were run and their outcomes.

Do **not** tick checkboxes speculatively or to “reserve” a task.

---

## 7. Presenting changes and diffs

When you present changes to a user (or as PR content), you must include:

1. **Task details**
   - Task ID and title (e.g., `M0-003: Wire AI coding guidelines to actual repo structure`).
   - Milestone (e.g., `M0`).

2. **Files touched**
   - List all modified files.
   - For each file, state:
     - Path.
     - Freeze state at time of modification (`mutable`, `review_only`, `frozen`—you must not touch `frozen`).
     - A short description of what changed.

3. **Tool runs**
   - Explicitly list commands executed, for example:
     - `cargo fmt`
     - `cargo test`
     - `cargo clippy --all-targets --all-features -- -D warnings`
     - `cargo fuzz run ingest_parser`
     - `cargo llvm-cov`
     - `cargo audit`
     - `cargo deny check`
     - `cargo udeps`
     - `cargo geiger`
     - …
   - For each, state whether it:
     - Passed.
     - Failed (with a short summary and link to where issues were fixed or documented).

4. **Behavioral summary**
   - Describe any new behavior or changes.
   - Note any remaining limitations or TODOs (and suggest new tasks if needed).

---

## 8. Alignment with human contributors

AI tools are expected to behave like careful junior developers under strict review.

- Mirror `CONTRIBUTING.md`:
  - Run the same “before PR” checks (core + extended tools as required).
  - Respect freeze states and coding standards.
- Do not attempt to bypass or weaken rules:
  - If a user asks you to “just ignore clippy” or “edit a frozen file,” you must:
    - Explain the conflict.
    - Offer a safer, compliant alternative.
    - Proceed only if the instruction clearly falls within what `.aicode` and `docs/rules.md` allow—and never for `frozen` files.

---

## 9. Summary checklist for AI tools

Before doing anything:

- [ ] Read `README.md` and `.aicode`.
- [ ] Read `docs/rules.md`, `docs/freeze.md`, `docs/roadmap.md`, `docs/milestones.md`, and `docs/tasks.md`.
- [ ] Skim `docs/architecture.md` and `docs/README.md`.

For each task:

- [ ] Select a single task from `docs/tasks.md` (or use user‑specified ID).
- [ ] Check freeze state for all files you intend to touch.
- [ ] Write or update tests first (or in tight lockstep).
- [ ] Implement the minimal code change.
- [ ] Run **all** relevant tools, even if some fail, to gather full diagnostics:
  - [ ] `cargo fmt`
  - [ ] `cargo test`
  - [ ] `cargo clippy --all-targets --all-features -- -D warnings`
  - [ ] Relevant `cargo fuzz run <target>`
  - [ ] `cargo llvm-cov` for touched modules
  - [ ] When deps/APIs/critical logic change:
    - [ ] `cargo audit`
    - [ ] `cargo deny check`
    - [ ] `cargo udeps`
    - [ ] `cargo machete`
    - [ ] `cargo outdated`
    - [ ] `cargo minimal-versions`
    - [ ] `cargo msrv`
    - [ ] `cargo vet`
    - [ ] `cargo supply-chain`
    - [ ] `cargo geiger`
    - [ ] `cargo semver-checks`
    - [ ] SBOM tools (`cargo about`, `cargo sbom`, `cargo cyclonedx`) when relevant
- [ ] Use all tool outputs to fix issues and add tests as needed.
- [ ] Re‑run tools until they are clean or exceptions are explicitly accepted by a human.
- [ ] Summarize changes, files, freeze states, and tool outcomes.
- [ ] Only then change the task’s checkbox from `[ ]` to `[x]` in `docs/tasks.md`.

AI tools must treat this document as a **mandatory procedure** for all work in the SentinelWX repository, not as optional advice.
