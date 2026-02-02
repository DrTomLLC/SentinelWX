# Contributing to SentinelWX

SentinelWX is a safety‑critical weather situational awareness and operations tool.  
Any change to this repository must be made with extreme care, strong tests, and full respect for the documented rules, freeze model, and extended tooling.

If existing code contradicts the rules or docs, **the docs win**.  
When in doubt, ask before making broad or risky changes.

---

## 1. Before you start

Please read these documents first:

- Root:
  - `README.md` – Project overview, goals, layout, and high‑level tooling philosophy.
  - `.aicode` – Primary instructions for AI coding assistants (also useful to understand control expectations).
- Under `docs/`:
  - `docs/rules.md` – Non‑negotiable rules (safety, coding standards, tests‑first, core + extended tooling).
  - `docs/roadmap.md` – Milestone‑based roadmap (M0–M8).
  - `docs/milestones.md` – Current milestone status and linkage to tasks.
  - `docs/tasks.md` – Concrete tasks (IDs, files to touch, acceptance criteria).
  - `docs/freeze.md` – Freeze policy (`mutable`, `review_only`, `frozen`) and central registry.
  - `docs/architecture.md` – High‑level architecture and design.
  - `docs/ai_coding_guidelines.md` – How AI tools are expected to operate, including full tooling loops and failure behavior.
  - `docs/README.md` – Documentation index and roles.

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
   - Writes/updates tests first or in lockstep with code changes.
   - Runs the full tooling loop described in `docs/ai_coding_guidelines.md`:
     - Core tools on **every** code/test change:
       - `cargo fmt`
       - `cargo test`
       - `cargo clippy --all-targets --all-features -- -D warnings`
       - Relevant `cargo fuzz run <target>`
       - `cargo llvm-cov` for touched modules
     - Extended tools when dependencies, public APIs, or safety‑critical modules are affected, such as:
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
       - Mutation/safety tools (`cargo mutants`, `cargo careful`) where required
   - Continues running all relevant tools even if one fails, to gather full diagnostics, then fixes issues and iterates until everything is clean.
   - Respects freeze states:
     - Never edits `frozen` files.
     - Only edits `review_only` files with explicit, narrow instructions.

Review all AI‑generated diffs as if they came from a junior contributor.

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
  - Keep changes minimal and well‑justified.
- When changing freeze state:
  - Update both:
    - The registry in `docs/freeze.md`.
    - The in‑file `SENTINELWX-FREEZE-STATE` marker.
  - Run the full core tooling loop (see below) to ensure stability.

---

## 4. Tasks, milestones, and scope

Work should be aligned with the roadmap and tracked as tasks:

- `docs/roadmap.md`  
  Describes milestones M0–M8 and their goals.
- `docs/milestones.md`  
  Tracks milestone status (`Not started / In progress / Complete`).
- `docs/tasks.md`  
  Lists concrete tasks by milestone, each with:
  - An ID (e.g., `M0-001`)
  - A short description
  - “Files to touch”
  - Acceptance criteria (including tooling expectations)

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

## 5. Tests, core tooling, and extended tooling – required checks

SentinelWX uses a strict tests‑first and tooling‑clean policy.

### 5.1 Tests

- Add or update tests for any behavioral change or new feature.
- Place tests according to project conventions:
  - Unit tests near the code (e.g., in the same module).
  - Integration tests in `tests/`.
- Tests must be:
  - Deterministic.
  - Free of external network dependencies.
  - Stable under repeated runs.

### 5.2 Core tools (always run for code/test changes)

For any PR that changes code or tests, you must run **all** of:

sh
cargo fmt
cargo test
cargo clippy --all-targets --all-features -- -D warnings
# For each relevant fuzz target:
cargo fuzz run <target>
cargo llvm-cov
cargo fmt

Code must be formatted before committing.

cargo test

All tests must pass.

cargo clippy --all-targets --all-features -- -D warnings

All warnings are treated as errors.

Prefer fixes over allow pragmas; any allow must be narrow and justified.

cargo fuzz run <target>

Run for fuzz targets that exercise code you changed.

Any crashes or unexpected behavior must be fixed and covered by tests.

cargo llvm-cov

Use coverage to ensure:

No regressions on safety‑critical paths.

New code is meaningfully covered.

You may also use cargo nextest run as an additional test runner, but it does not replace cargo test in expectations.

5.3 Extended tools (when deps, APIs, or critical logic change)
If your change modifies dependencies, Cargo.toml/Cargo.lock, public APIs, or safety‑critical modules, you must also run all relevant extended tools:

Dependency and supply‑chain:

text
cargo audit
cargo deny check
cargo udeps
cargo machete
cargo outdated
cargo minimal-versions
cargo msrv
cargo vet
cargo supply-chain
cargo about   # as configured
cargo sbom    # as configured
cargo cyclonedx   # as configured
cargo audit

Failures (advisories, yanked crates) must be fixed or explicitly waived by maintainers.

cargo deny check

Enforce license and source policies; violations must be addressed or explicitly accepted.

cargo udeps and cargo machete

Remove unused dependencies and features or justify their presence.

cargo outdated

Review outdated crates; for upgrade PRs, update where reasonable.

cargo minimal-versions and cargo msrv

Ensure minimal and MSRV builds still work.

cargo vet and cargo supply-chain

Inspect and vet new or sensitive dependencies.

SBOM tools (cargo about, cargo sbom, cargo cyclonedx)

Generate license and SBOM artifacts for releases or compliance‑focused changes.

Safety and API stability:

text
cargo geiger
cargo semver-checks
cargo careful    # where applicable
cargo mutants    # where applicable
cargo geiger

Detect unsafe usage; new unsafe is strongly discouraged and must be justified and reviewed.

cargo semver-checks

Ensure no unintended breaking changes when public APIs are affected.

cargo careful

For critical codepaths, used to catch UB‑like issues.

cargo mutants

For mutation testing on safety‑critical logic; surviving mutants must be addressed or documented.

5.4 Do not stop at the first failure
Important:

Do not stop the check process when the first tool fails.

Even if, for example, cargo test fails:

Continue to run the remaining core and relevant extended tools.

Collect all diagnostics from all tools.

Only then:

Plan fixes based on the full set of issues.

Apply minimal, well‑tested changes.

Re‑run the relevant tools until they are clean or exceptions are explicitly accepted by maintainers.

This applies to both humans and AI tools.

5.5 “Before PR” checklists
Always (for any code/test change):

 cargo fmt

 cargo test

 cargo clippy --all-targets --all-features -- -D warnings

 All relevant cargo fuzz run <target> commands

 cargo llvm-cov for modules you touched

If you changed dependencies, Cargo.toml/Cargo.lock, public APIs, or safety‑critical modules:

 cargo audit

 cargo deny check

 cargo udeps

 cargo machete

 cargo outdated

 cargo minimal-versions

 cargo msrv

 cargo vet (where configured)

 cargo supply-chain (review for concerning results)

 cargo geiger

 cargo semver-checks

 SBOM tools (cargo about, cargo sbom, cargo cyclonedx) as appropriate

 cargo careful and/or cargo mutants when required by relevant tasks/milestones

Run the full set that applies, even if one command fails, then fix issues and re‑run until clean.

PRs that skip required tools or ignore their failures may be rejected or asked to be revised.

6. Coding standards and safety expectations
Key expectations (see docs/rules.md for full details):

Rust

Favor clear, idiomatic Rust over clever tricks.

Avoid unsafe unless explicitly justified and approved.

Avoid panics in normal operation; errors should be handled or clearly propagated.

Safety and security

Never log secrets or sensitive data.

Treat all external and user input as untrusted.

Favor conservative, explicit error handling.

Documentation

Keep relevant docs in sync with behavior:

docs/architecture.md

docs/roadmap.md

docs/rules.md

README.md

docs/tasks.md (if task scope or acceptance criteria change)

If a change would materially alter architecture or roadmap, coordinate with maintainers first.

7. Submitting changes
Fork and branch

Fork the repository.

Create a feature branch with a descriptive name.

Optionally reference the task ID (e.g., feature/M1-001-config-scaffold).

Implement and test

Work against a specific task from docs/tasks.md when possible.

Keep commits small and logically grouped.

Run the full applicable tooling sets described above.

Do not stop running tools at the first failure; gather all issues, then iterate.

Open a pull request

Describe:

Which task(s) you addressed (ID and title).

Files you changed and their freeze states (especially for any review_only ones).

Any changes to freeze states (and reference updates to docs/freeze.md and in‑file markers).

Include:

Confirmation that you ran:

Core tools (cargo fmt, cargo test, cargo clippy --all-targets --all-features -- -D warnings, relevant cargo fuzz run <target>, cargo llvm-cov).

Extended tools for dependency/API/critical‑module changes.

A summary of any remaining limitations or TODOs.

Respond to review

Be prepared to:

Update tests, code, or docs per feedback.

Rerun tools after fixes.

Keep discussions focused on safety, correctness, and clarity.

8. Reporting issues and security concerns
For general bugs:

Open an issue including:

A clear description of the problem.

Steps to reproduce.

Environment details (OS, Rust version, how you ran SentinelWX).

Relevant logs, test output, or tool diagnostics.

The milestone/task you believe it affects (if known).

For security‑sensitive issues:

Do not open a public issue.

Use the private contact method described in docs/rules.md (once defined) or the maintainer’s preferred secure channel.

Provide enough detail for reproduction and triage, but avoid sharing secrets or operationally sensitive information in plain text.

9. Maintainer responsibilities
Maintainers are responsible for:

Keeping docs/ (architecture, roadmap, rules, freeze, tasks, milestones, AI guidelines) accurate and consistent.

Managing freeze states:

Deciding when to move files between mutable, review_only, and frozen.

Ensuring markers and docs/freeze.md stay in sync.

Curating and updating tasks in docs/tasks.md.

Enforcing:

Tests‑first and clear coding standards.

Strict core and extended tooling on code changes.

Safety and security standards.

They may:

Reject or request changes to PRs that:

Skip required tools or ignore failures.

Violate freeze rules.

Introduce unsafe or unclear behavior.

Drift from architecture, roadmap, or control docs.

By contributing to SentinelWX, you agree to follow these guidelines and treat this project as safety‑critical software where every tool run and every test matters.
