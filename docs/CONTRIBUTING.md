# Contributing to SentinelWX

SentinelWX is a safety‑critical weather situational awareness and operations tool.  
Any change to this repository must be made with extreme care, strong tests, and full respect for the documented rules and freeze model.

If existing code contradicts the rules or docs, **the docs win**.  
When in doubt, ask before making broad or risky changes.

---

## 1. Before you start

Please read these documents first:

- Root:
  - `README.md` – project overview, goals, and layout.
  - `.aicode` – primary instructions for AI coding assistants (also useful to understand control expectations).
- Under `docs/`:
  - `docs/rules.md` – non‑negotiable rules (safety, security, coding, tests‑first).
  - `docs/roadmap.md` – milestone‑based roadmap (M0–M8).
  - `docs/milestones.md` – current milestone status and linkage to tasks.
  - `docs/tasks.md` – concrete tasks (IDs, files to touch, acceptance criteria).
  - `docs/freeze.md` – freeze policy (`mutable`, `review_only`, `frozen`) and central registry.
  - `docs/architecture.md` – high‑level architecture and design.
  - `docs/ai_coding_guidelines.md` – how AI tools are expected to operate in this repo.

These documents are the source of truth for how SentinelWX is developed and maintained.

---

## 2. Using AI coding assistants

You may use AI coding assistants (e.g., Claude Code, Antigravity), but:

- You remain responsible for all changes.
- AI tools must **obey**:
  - `.aicode`
  - `docs/rules.md`
  - `docs/freeze.md`
  - `docs/tasks.md`
  - `docs/ai_coding_guidelines.md`

If you use AI:

1. Make sure your tool loads `.aicode` and the docs listed above.
2. Drive AI work from explicit tasks in `docs/tasks.md` (one task at a time).
3. Ensure the AI:
   - Writes/updates tests first.
   - Runs the full command loop:
     - `cargo test`
     - `cargo clippy --all-targets --all-features -- -D warnings`
     - `cargo fuzz run <target>` for relevant fuzz targets, when they exist.
   - Respects freeze states:
     - Never edits `frozen` files.
     - Only edits `review_only` files with explicit, narrow instructions.
4. Review all AI‑generated diffs as if they came from a junior contributor.

Do **not** merge AI‑generated changes that ignore the rules or skip required checks.

---

## 3. Freeze model and when you may edit files

SentinelWX uses a file freeze model defined in `docs/freeze.md`:

- `mutable`
  - Normal development; files can be edited when working on a relevant task.
- `review_only`
  - Stable files; maintainers may make small, deliberate edits.
  - AI tools must not edit these files unless explicitly instructed for a narrow change.
- `frozen`
  - Files that are considered complete and must not change under normal circumstances.

Before editing any file:

1. Check `docs/freeze.md` for its current state.
2. Look for an in‑file `SENTINELWX-FREEZE-STATE` marker near the top.

Rules:

- Do not edit `frozen` files unless:
  - You are a maintainer, and
  - You have decided to unfreeze or downgrade the file state, following the process in `docs/freeze.md`.
- Be cautious editing `review_only` files:
  - Prefer to coordinate changes through maintainers.
  - Keep changes minimal and well justified.
- When changing freeze state:
  - Update both:
    - The registry in `docs/freeze.md`.
    - The in‑file `SENTINELWX-FREEZE-STATE` marker.
  - Run the full test/Clippy/fuzz loop (see below).

---

## 4. Tasks, milestones, and scope

Work should be aligned with the roadmap and tracked as tasks:

- `docs/roadmap.md`
  - Describes milestones M0–M8 and their goals.
- `docs/milestones.md`
  - Tracks milestone status (`Not started / In progress / Complete`).
- `docs/tasks.md`
  - Lists concrete tasks by milestone, each with:
    - An ID (e.g., `M0-001`),
    - A short description,
    - “Files to touch,”
    - Acceptance criteria.

Guidelines:

- Prefer to choose an existing task from `docs/tasks.md` and work on that.
- If you need a new task:
  - Discuss it with a maintainer.
  - Have a maintainer add or adjust tasks in `docs/tasks.md`.
- Keep each change focused on **one task**:
  - Avoid large, multi‑task PRs.
  - Avoid drive‑by refactors unrelated to the current task.

Only human maintainers should:

- Add or remove milestones.
- Change milestone status.
- Rewrite tasks or change task IDs.

---

## 5. Tests, Clippy, and fuzzing – required checks

SentinelWX uses a strict tests‑first and lint‑clean policy:

### 5.1 Tests

- Add or update tests for any behavioral change or new feature.
- Place tests according to project conventions:
  - Unit tests near the code (e.g., in the same module).
  - Integration tests in `tests/`.
- Tests must be:
  - Deterministic.
  - Free of external network dependencies.
  - Stable under repeated runs.

### 5.2 Clippy

You must run:

sh
cargo clippy --all-targets --all-features -- -D warnings
All warnings are treated as errors.

Fix issues rather than silencing them.

If you must allow a lint:

Use the narrowest possible scope.

Add a short comment explaining why the allowance is necessary.

5.3 Fuzzing
Where fuzz targets exist (for parsers, data conversions, etc.), you must run:

text
cargo fuzz run <target>
for each target that exercises code you have touched.

If fuzzing finds a crash or unexpected behavior:

Treat it as a bug.

Fix the underlying issue.

Add or update tests to prevent regressions.

5.4 “Before PR” checklist
Before opening a PR, ensure that:

 All relevant tests compile and pass:

cargo test

 Clippy passes without warnings:

cargo clippy --all-targets --all-features -- -D warnings

 All relevant fuzz targets run clean (where applicable):

cargo fuzz run <target> for each affected target

 Changes respect:

docs/rules.md

docs/freeze.md

docs/architecture.md

docs/tasks.md

 Any changes to freeze state follow the documented process in docs/freeze.md.

PRs that fail any of these checks may be rejected or asked to be revised.

6. Coding standards and safety expectations
Key expectations (see docs/rules.md for full details):

Rust

Favor clear, idiomatic Rust over clever tricks.

Avoid unsafe unless explicitly justified and approved.

Avoid panics in normal operation; errors should be handled or propagated clearly.

Safety and security

Never log secrets or sensitive data.

Be conservative about default behaviors and error handling.

Treat user input and external data as untrusted.

Documentation

Keep relevant docs in sync with behavior:

docs/architecture.md

docs/roadmap.md

docs/rules.md

README.md

If a change would materially alter architecture or roadmap, coordinate with maintainers first.

7. Submitting changes
Fork and branch

Fork the repository.

Create a feature branch with a descriptive name.

Implement and test

Work against a specific task from docs/tasks.md when possible.

Keep commits small and logically grouped.

Run the full test/Clippy/fuzz suite as described above.

Open a pull request

Describe:

Which task(s) you addressed (ID and title).

Files you changed.

Any changes to freeze states.

Include:

Confirmation that you ran:

cargo test

cargo clippy --all-targets --all-features -- -D warnings

Relevant cargo fuzz run <target> commands.

Any limitations or follow‑up work you see.

Respond to review

Be prepared to update tests, code, or docs per feedback.

Keep discussions focused on safety, correctness, and clarity.

8. Reporting issues and security concerns
For general bugs:

Open an issue including:

A clear description of the problem.

Steps to reproduce.

Environment details (OS, Rust version, how you ran SentinelWX).

Any logs or test output that help diagnose the issue.

The milestone/task you believe it affects (if known).

For security‑sensitive issues:

Do not open a public issue.

Use the private contact method described in docs/rules.md (once defined) or the maintainer’s preferred secure channel.

Provide enough detail for reproduction and triage, but avoid sharing secrets or operationally sensitive information in plain text.

9. Maintainer responsibilities
Maintainers are responsible for:

Keeping docs/ (architecture, roadmap, rules, freeze, tasks, milestones) accurate and consistent.

Managing freeze states:

Deciding when to move files between mutable, review_only, and frozen.

Ensuring markers and docs/freeze.md stay in sync.

Curating and updating tasks in docs/tasks.md.

Enforcing:

Tests‑first, lint‑clean, fuzz‑clean policies.

Strict adherence to docs/rules.md.

They may also:

Reject or request changes to PRs that:

Skip tests, Clippy, or fuzzing.

Violate freeze rules.

Introduce unsafe or unclear behavior.

By contributing to SentinelWX, you agree to follow these guidelines and treat this project as safety‑critical software.
