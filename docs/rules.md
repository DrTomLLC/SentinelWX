# SentinelWX rules and non‑negotiables

These rules are mandatory for all contributors and all AI tools working in this repository.  
If existing code appears to conflict with these rules, **the rules win** and the code must be brought into alignment.

SentinelWX is treated as **safety‑critical** software.  
Users are assumed to rely on it for real operational decisions (incident command, emergency management, serious weather operations).  
Sloppy changes, speculative refactors, and “quick hacks” are not acceptable.

If you are unsure whether a change violates these rules, **stop and ask** before proceeding.

---

## 1. Safety, reliability, and scope

1. SentinelWX must behave predictably under stress.
   - No panics in normal operation.
   - No silent data corruption.
   - No unbounded resource usage without clear limits and backpressure.

2. Safety beats convenience.
   - Prefer explicit, defensive code over cleverness.
   - Handle error cases explicitly; do not swallow errors.

3. Minimal scope.
   - Each change should address a specific, documented need:
     - A task in `docs/tasks.md`, or
     - A clearly documented bug / requirement.
   - Avoid scope creep and opportunistic refactors.

4. Architecture is not optional.
   - All changes must respect `docs/architecture.md` and `docs/roadmap.md`.
   - If a change requires architectural adjustments, update the docs **first or alongside** the code.

---

## 2. Language and coding standards (Rust)

1. Use idiomatic Rust.
   - Favor clarity over clever tricks.
   - Avoid macro abuse unless it clearly improves safety or eliminates duplication.

2. `unsafe` is effectively banned unless explicitly justified.
   - New `unsafe` code requires:
     - A clear, documented rationale.
     - A short explanation in comments near the block.
     - Review from a maintainer.
   - All `unsafe` usage must show up in `cargo geiger` output and must be **intentional** and reviewed.[web:102]

3. Error handling.
   - Prefer `Result` and well‑typed errors over panics.
   - Use `panic!` only for programmer errors that should never occur in production (and even then, prefer not to).
   - Propagate errors upward when appropriate; do not silently ignore them.

4. Logging and observability.
   - Logging must not leak secrets or sensitive data.
   - Logged messages must be useful under stress (concise, clear, structured where reasonable).
   - Do not rely on logging as the sole mechanism for correctness; tests must assert behavior.

5. Style.
   - Use `cargo fmt` for code formatting.
   - Follow the project’s existing patterns and conventions.

---

## 3. Tests‑first and verification philosophy

All behavior changes and new features must be test‑driven.

1. Tests come first (or in tight lockstep).
   - For any new behavior:
     - Add or update tests to express the desired behavior.
     - Prefer to make tests fail in the intended way before implementing code.
   - Exceptions (e.g., mechanical refactoring) must still be covered by existing tests.

2. Types of tests.
   - Unit tests near the modules they exercise.
   - Integration tests in `tests/` for cross‑module and end‑to‑end behavior.
   - Fuzz tests for parsers, data transformations, and other complex logic that must be robust.

3. Determinism.
   - Tests must be deterministic and repeatable.
   - Do not depend on external network services or nondeterministic timing.

4. Coverage.
   - Use `cargo llvm-cov` (or equivalent LLVM coverage) to monitor coverage levels, especially for safety‑critical code paths.[web:107][web:119][web:108]
   - When you touch a module:
     - Do not reduce its meaningful coverage.
     - Prefer to increase coverage for critical logic.

5. Mutation testing (targeted).
   - For safety‑critical modules, `cargo mutants` may be required as a task acceptance criterion.[web:102]
   - When mutation testing is used:
     - Surviving mutants must be investigated.
     - Either:
       - Add tests that kill them, or
       - Document and justify why the equivalent behavior is acceptable.

---

## 4. Tooling and analysis (core loop + extended tools)

SentinelWX uses an **extended** toolchain.  
The default expectation is: **run these tools, fix every issue they find, write tests where necessary, and repeat until clean.**[web:93][web:99][web:95]

### 4.1 Always‑run tools (per task / per PR)

For any change that affects code or tests:

1. `cargo fmt`
   - Code must be formatted before committing or opening a PR.

2. `cargo test`
   - Run the full test suite, unless there is a documented reason to narrow scope.
   - All tests must pass.

3. `cargo clippy --all-targets --all-features -- -D warnings`
   - All warnings are treated as errors.[web:32]
   - Fix issues rather than silencing them.
   - If you must allow a lint:
     - Use the narrowest possible scope (`#[allow(...)]`).
     - Add a short comment explaining why.

4. `cargo fuzz run <target>`
   - For each fuzz target relevant to changed code:
     - Run `cargo fuzz run <target>`.
   - Any crashes or unexpected behavior must be fixed and covered by tests.[web:110]

5. `cargo llvm-cov`
   - Use `cargo llvm-cov` (or equivalent) to check coverage for modules you touched.[web:107][web:119][web:108]
   - Do not regress coverage on safety‑critical paths.
   - For new modules, aim for meaningful coverage, not just raw percentages.

6. `cargo nextest run` (optional but recommended for larger suites)
   - May be used as a faster test runner, but does **not** replace `cargo test` in CI expectations.

These tools define the **core loop** that must be clean before a task in `docs/tasks.md` can be marked complete.

### 4.2 Dependency and supply‑chain tools (run when deps change or periodically)

When you modify `Cargo.toml`, `Cargo.lock`, features, or add/remove dependencies, you must treat supply‑chain and license risk as part of the change.[web:93][web:99][web:95][web:118]

Run:

1. `cargo audit`
   - Command: `cargo audit -D` or `cargo audit --deny-warnings`.[web:95][web:96][web:115]
   - Any advisory or yanked crate is a **failure** until:
     - The dependency is updated or removed, or
     - The risk is explicitly documented and accepted by maintainers.

2. `cargo deny check`
   - Use `cargo deny` to enforce:
     - License policy (e.g., no incompatible licenses).
     - Sources and advisory policy.[web:118]
   - Failures must be fixed or explicitly waived by maintainers.

3. `cargo udeps`
   - Detect unused dependencies.
   - Remove unused entries from `Cargo.toml` or justify why they remain.[web:93][web:94]

4. `cargo machete`
   - Detect unused features and tighten feature flags.

5. `cargo outdated`
   - Identify outdated dependencies.
   - For upgrade work:
     - Bring selected crates up to date.
     - Ensure tests and extended tools remain clean.
   - It is acceptable to leave some crates outdated, but choices must be deliberate and documented in PRs or issues.[web:99]

6. `cargo minimal-versions`
   - Validate that the project builds with minimal compatible versions.
   - Helps enforce MSRV and compatibility guarantees.

7. `cargo msrv`
   - Check the minimum supported Rust version, especially when dependencies or language features change.

8. `cargo vet`
   - Use for high‑assurance dependency vetting.
   - New dependencies should pass `cargo vet` policy where configured.

9. `cargo supply-chain`
   - Inspect who maintains dependencies and where they come from.
   - For safety‑critical components, maintainers may require explicit review of `cargo supply-chain` output.

10. SBOM and license reporting:
    - `cargo about`, `cargo sbom`, `cargo cyclonedx`
    - Used primarily for releases and milestone work that touches compliance.
    - Generated SBOMs and license reports must be stored or referenced as part of release artifacts.

### 4.3 Safety and API stability tools

For safety‑critical modules and public APIs:

1. `cargo geiger`
   - Run to detect `unsafe` usage.
   - New `unsafe` blocks are not allowed without explicit maintainer review and justification.[web:102]

2. `cargo semver-checks`
   - When versioning artifacts or changing public APIs:
     - Run `cargo semver-checks` to ensure no unintended breaking changes.
   - Breaking changes must be intentional, documented, and aligned with `docs/roadmap.md`.

3. `cargo careful`
   - For certain tasks, maintainers may require `cargo careful` builds/tests to catch UB‑like issues.
   - Failures must be treated as bugs and fixed.

4. `cargo mutants`
   - Mutation testing for critical logic paths.
   - Used selectively as part of hardening milestones.
   - Surviving mutants require new tests or explicit, documented justification.

### 4.4 Failure semantics

By default:

- Any failure or warning from:
  - `cargo fmt`
  - `cargo test`
  - `cargo clippy --all-targets --all-features -- -D warnings`
  - `cargo fuzz run <target>`
  - `cargo llvm-cov` (coverage regressions on critical paths)
  - `cargo audit`, `cargo deny`, `cargo udeps`, `cargo geiger`, `cargo semver-checks`
- Must be treated as a **blocker** until:
  - The underlying issue is fixed (with tests as needed), or
  - Maintainers explicitly decide to accept a documented risk or limitation.

Tools like `cargo outdated`, `cargo minimal-versions`, `cargo msrv`, `cargo vet`, SBOM generators, and mutation testing are:
- **Required** when specified in task acceptance criteria or milestone descriptions.
- Otherwise, they are **strongly recommended** and should be run regularly by maintainers.

---

## 5. Freeze model and file states

Freezing is how we protect stable or safety‑critical areas from accidental edits, especially by AI tools.  
The authoritative description lives in `docs/freeze.md`.

Key principles:

1. Only humans (maintainers) decide when to:
   - Mark files as `mutable`, `review_only`, or `frozen`.
   - Change a file’s freeze state.

2. AI tools must:
   - Never change freeze states.
   - Never edit `frozen` files.
   - Only edit `review_only` files when explicitly instructed for a narrow change.

3. Every file that participates in the freeze model must have:
   - A registry entry in `docs/freeze.md` (at least for `review_only` and `frozen`).
   - An in‑file `SENTINELWX-FREEZE-STATE` marker.

4. When freeze state changes:
   - Update both the registry and the in‑file marker.
   - Run the core tool loop (tests, clippy, fuzz, etc.) to ensure stability.

For full details, see `docs/freeze.md`.

---

## 6. Tasks, milestones, and roadmap discipline

Work must be driven by documented tasks and milestones:

1. Roadmap and milestones.
   - `docs/roadmap.md` defines milestones M0–M8 and high‑level goals.
   - `docs/milestones.md` provides:
     - A short goal sentence per milestone.
     - Status (`Not started / In progress / Complete`).
     - A pointer to `docs/tasks.md`.

2. Tasks.
   - `docs/tasks.md` is the single source of truth for concrete tasks:
     - Each task has an ID (e.g., `M0-001`), description, “files to touch,” and acceptance criteria.
   - Tasks must:
     - Refer back to milestones and roadmap where appropriate.
     - Include acceptance criteria that require:
       - Tests updated/added.
       - Core tools clean (`cargo fmt`, `cargo test`, `cargo clippy`, relevant fuzz).
       - Extended tools clean when dependencies, public APIs, or safety‑critical modules are affected.

3. One task at a time.
   - Humans and AI tools should focus changes on one task per PR/branch where practical.
   - Avoid mixing unrelated tasks.

4. Who manages tasks and milestones.
   - Only maintainers may:
     - Add/remove milestones.
     - Change milestone status.
     - Re‑scope tasks or change IDs.
   - Contributors may propose new tasks or changes via issues/PRs, but maintainers decide what becomes canonical.

---

## 7. AI responsibilities and constraints

AI coding assistants (Claude Code, Antigravity, etc.) must obey all of the above, plus the additional instructions in `.aicode` and `docs/ai_coding_guidelines.md`.

Key rules:

1. Source of truth.
   - `.aicode` is the primary AI instruction file.
   - `docs/rules.md`, `docs/freeze.md`, `docs/tasks.md`, and `docs/ai_coding_guidelines.md` are binding.
   - If in doubt, the AI must assume the strictest rule.

2. Task‑driven.
   - AI tools must:
     - Pick tasks from `docs/tasks.md` (or use an explicit user‑specified ID).
     - Work on one task at a time.
     - Not invent new tasks or rewrite IDs on their own.

3. Freeze respect.
   - AI must:
     - Check `docs/freeze.md` and in‑file markers before editing.
     - Never edit `frozen` files.
     - Only edit `review_only` files when explicitly instructed for a narrow change.
   - If a task requires changing a protected file:
     - AI must call this out and wait for a human to adjust freeze state or make the change.

4. Tool loop.
   - AI must:
     - Implement the tests‑first, minimal‑change approach.
     - Run the core tools:
       - `cargo fmt`
       - `cargo test`
       - `cargo clippy --all-targets --all-features -- -D warnings`
       - Relevant `cargo fuzz run <target>`
       - `cargo llvm-cov` for coverage on touched modules
     - When dependencies/public APIs/safety‑critical modules change, run extended tools as defined in section 4.
     - Fix issues and iterate until everything is clean.
     - Only then mark tasks as complete.

5. Reporting.
   - AI must:
     - List the task ID.
     - List files changed and their freeze state.
     - State which tools were run and whether they passed.
     - Highlight any remaining warnings/limitations.

6. Refusal conditions.
   - AI must refuse to:
     - Edit frozen files.
     - Ignore failing tests, clippy, fuzz, or extended tools.
     - Perform broad refactors or architectural changes without explicit instruction.

---

## 8. Human maintainer responsibilities

Maintainers are responsible for:

1. Keeping `docs/` accurate and up to date.
2. Managing freeze states and ensuring they reflect reality.
3. Curating tasks and milestones.
4. Enforcing:
   - Tests‑first development.
   - Strict tooling (core + extended) on code changes.
   - Safety and security standards.

They may:

- Reject or request changes to PRs that:
  - Skip required tools.
  - Violate freeze rules.
  - Introduce unsafe or unclear behavior.
  - Drift from architecture and roadmap.

---

These rules exist to ensure SentinelWX remains trustworthy, predictable, and suitable for life‑critical operational use.  
All contributors—human and AI—are expected to follow them strictly.
