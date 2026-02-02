# Freeze policy and file registry

SentinelWX is treated as **safety‑critical**.  
This document defines the freeze model for source files and docs, and how humans and AI tools must behave with respect to frozen and review‑only areas.

If anything in existing code or documentation conflicts with this file, **this file wins**.  
Humans may refine this policy, but AI tools must never change it unless explicitly instructed to do so.

## 1. Roles and scope

- This policy applies to:
  - All files in the repository (source, tests, docs, configs, CI, tooling).
  - All contributors, including humans and AI tools.
- Freeze decisions:
  - Only humans (maintainers) decide when to freeze or unfreeze.
  - AI tools must never autonomously freeze, unfreeze, or change freeze status.
- Source of truth:
  - This document (`docs/freeze.md`).
  - The freeze registry table in section 4.
  - In‑file `SENTINELWX-*` markers embedded in individual files.

If there is any disagreement between a file’s current contents and the freeze registry, humans must resolve it; AI tools must treat the stricter interpretation as binding.

## 2. Freeze states

Every file may be in exactly one of the following states:

- `mutable`
- `review_only`
- `frozen`

These states are interpreted as follows.

### 2.1 `mutable`

- Meaning:
  - The file is open for normal development.
  - Both humans and AI tools may modify the file when working on explicit tasks.
- Rules:
  - Changes must still obey `docs/rules.md`, `.aicode`, and the tests/clippy/fuzz requirements.
  - Large or risky changes should be split into small, well‑tested steps.
  - When a file becomes stable enough, a maintainer may move it to `review_only` or `frozen`.

### 2.2 `review_only`

- Meaning:
  - The file is considered stable and should not change frequently.
  - Human maintainers may make small, deliberate edits (typos, clarifications, minor fixes).
  - AI tools must **not** edit `review_only` files unless explicitly instructed by a human for a specific, narrow change.
- Rules:
  - Default behavior for AI:
    - Do not propose or apply edits to `review_only` files.
    - If a task would require changing a `review_only` file, the AI should:
      - Call this out in its explanation.
      - Wait for a human to either:
        - Temporarily move the file back to `mutable`, or
        - Perform the edit themselves.
  - Before moving a file from `mutable` to `review_only`, maintainers should:
    - Ensure tests for that file’s behavior are comprehensive and passing.
    - Ensure clippy and fuzz (where applicable) are clean.
    - Confirm consistency with `docs/rules.md` and the architecture.

### 2.3 `frozen`

- Meaning:
  - The file is considered **complete and untouchable** under normal circumstances.
  - Both AI tools and human contributors must treat it as read‑only unless a maintainer explicitly unfreezes or grants an exception.
- Rules:
  - AI tools:
    - Must never modify `frozen` files.
    - Must not propose edits to `frozen` files unless a human explicitly requests changes and confirms that the file has been or will be unfrozen.
  - Human maintainers:
    - Should only unfreeze or edit `frozen` files when there is a strong, documented reason (e.g., bug in safety‑critical logic, security fix, regulatory requirement).
    - Must ensure all tests, clippy, and fuzzers are updated and passing after any change.
  - A file should not be marked `frozen` until:
    - It is covered by strong tests.
    - `cargo test` and `cargo clippy --all-targets --all-features -- -D warnings` pass clean.
    - Relevant fuzz targets run clean, where applicable.
    - It is consistent with `.aicode`, `docs/rules.md`, and `docs/architecture.md`.

## 3. How to freeze and unfreeze files (human process)

Only human maintainers may change freeze state.  
The steps below apply whether you are moving a file between `mutable`, `review_only`, and `frozen`.

### 3.1 Changing freeze state

To change the state of a file:

1. **Decide the new state.**
   - Choose `mutable`, `review_only`, or `frozen` based on the definitions in section 2.
2. **Update the registry in `docs/freeze.md`.**
   - Add or update the entry for the file in the table in section 4.
   - Commit this change as part of the same change set that adds/removes markers.
3. **Update in‑file markers.**
   - Open the target file and ensure its `SENTINELWX-*` markers match the new state (see section 3.2).
4. **Run checks.**
   - Run `cargo test`.
   - Run `cargo clippy --all-targets --all-features -- -D warnings`.
   - Run relevant `cargo fuzz run <target>` commands if fuzz harnesses exist for the affected area.
   - Only push/merge after all checks pass clean.
5. **Communicate to AI tools (if relevant).**
   - If you are using AI (Claude Code, Antigravity, etc.), ensure your prompt or `.aicode` context indicates that the freeze registry was updated.

### 3.2 In‑file markers

Each file that participates in the freeze model should contain a clear marker comment near the top of the file, using a format appropriate to the language:

- Rust source example:

  ```rust
  // SENTINELWX-FREEZE-STATE: mutable
Markdown / text example:

text
<!-- SENTINELWX-FREEZE-STATE: review_only -->
TOML / config example:

text
# SENTINELWX-FREEZE-STATE: frozen
Rules for markers:

The marker must specify one of: mutable, review_only, or frozen.

The marker must match the state recorded in the registry table.

When changing the freeze state, always update both:

The registry table.

The in‑file marker.

AI tools must treat the marker as authoritative and refuse to modify files marked as review_only or frozen unless explicitly instructed otherwise.

4. Freeze registry
This table is the central registry of freeze state for important files.
It does not need to list every file in the repo, but any file that is review_only or frozen must appear here. mutable entries are encouraged for clarity.

Maintain this table in sorted, readable form.
Paths are relative to the repo root.

Path	State	Notes
README.md	mutable	Root overview; keep in sync with docs.
CONTRIBUTING.md	mutable	Contributor guide.
.aicode	review_only	AI control; only maintainers should edit.
docs/README.md	mutable	Docs index.
docs/architecture.md	mutable	Architecture design.
docs/roadmap.md	review_only	Canonical roadmap; only maintainers adjust.
docs/rules.md	review_only	Core rules; changes must be deliberate.
docs/freeze.md	review_only	This file; maintainers control freeze policy.
docs/milestones.md	mutable	Milestone status connector.
docs/tasks.md	mutable	Task list and progress checkboxes.
docs/ai_coding_guidelines.md	mutable	AI‑facing guidelines.
Maintain this table as the single, readable summary of freeze states for high‑impact files.
For other files (e.g., individual Rust modules), you may add rows as they become stable enough to be review_only or frozen.

5. AI tool obligations
AI coding tools (Claude Code, Antigravity, etc.) must obey the following:

Never modify frozen files.

If a task appears to require changes to a frozen file, the AI must:

Say so explicitly in its reasoning or explanation to the user.

Request that a human either:

Unfreeze the file (by updating this document and the in‑file marker), or

Perform the change manually.

The AI must not attempt workarounds that effectively bypass the frozen logic (e.g., duplicating frozen functionality elsewhere) unless explicitly instructed.

Do not modify review_only files by default.

The AI may read review_only files freely.

It must not modify them unless:

The user explicitly instructs the AI to change that file, and

The instruction is narrow and specific (e.g., “fix this typo,” “update this version string”).

Even with explicit instructions, if the file is marked frozen, the AI must refuse until the human unfreezes it.

Treat the stricter rule as binding.

If there is any conflict between:

.aicode

docs/rules.md

This file (docs/freeze.md)

In‑file SENTINELWX-* markers

The AI must assume the most restrictive interpretation and avoid modifying the file until a human clarifies.

Respect tasks and scope.

The AI must not change freeze states as part of completing a task.

The AI must not “auto‑invent” tasks to change freeze states.

All AI work must be scoped to:

The selected task from docs/tasks.md, and

Files that are mutable for that task.

Report touched files.

When presenting changes, AI tools should list:

All files modified.

For each file, the current freeze state (as interpreted from this document and in‑file markers).

If any file modified is later moved to review_only or frozen, that should be done by humans following section 3.

6. Human maintainer guidance
Maintainers should use freezing strategically:

Use mutable for:

Early, exploratory development.

Areas under active iteration or refactoring.

Use review_only for:

Stabilized docs and design files.

Modules that are mature but may still see occasional, careful edits.

Use frozen for:

Fully validated, well‑tested, safety‑critical logic.

Documents that encode core project control (e.g., finalized .aicode variants or regulatory requirements).

Before freezing a file:

Confirm that tests cover its behavior well.

Confirm cargo test and cargo clippy --all-targets --all-features -- -D warnings pass.

Run relevant cargo fuzz run <target> commands when fuzz harnesses exist for that area.

Ensure the file is consistent with docs/rules.md, .aicode, and docs/architecture.md.

If an emergency fix requires modifying a frozen file:

Temporarily move the file to mutable or review_only following section 3.

Make the minimal, well‑tested change.

Re‑run tests, clippy, and fuzzers.

Once stable again, move it back to frozen and update the registry and markers.

This freeze model is designed so that once parts of SentinelWX are marked review_only or frozen, AI tools cannot “thrash” them while still being able to work productively on clearly mutable areas, guided by docs/tasks.md and .aicode.
