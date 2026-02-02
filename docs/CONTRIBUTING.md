# Contributing to SentinelWX

Thank you for your interest in contributing to SentinelWX.  
This is a safety‑critical project, and contributions must respect strict rules around reliability, data provenance, and security. [web:201][web:213]

## Ground rules

Before opening issues or PRs, you must read:

- `docs/rules.md`
- `docs/architecture.md`
- `docs/roadmap.md`
- `docs/milestones.md`
- `docs/tasks.md`
- `docs/freeze.md`
- `.aicode` (if you use AI tooling)

By contributing, you agree to follow these documents and keep them up to date when your changes affect them.

Key expectations:

- No panics in production paths.
- No `unsafe` without prior discussion and explicit justification.
- Exhaustive error handling with meaningful messages and logs.
- Clear separation between official data and derived/diagnostic products.
- Tests‑first mindset and a clean test/lint/fuzz state before merge. [web:206][web:211]

## Workflow

1. **Pick or define work**

   - Prefer tasks from `docs/tasks.md`.
   - If you need to add or change tasks, update `docs/tasks.md` and, if necessary, `docs/milestones.md` and `docs/roadmap.md`.

2. **Design**

   - Review `docs/architecture.md` and relevant sections before making structural changes.
   - For larger changes, consider proposing a brief design in an issue.

3. **Implement with tests**

   - Write or update tests for the behavior you are changing or adding.
   - Keep changes small and focused on a single task or closely related set of tasks.

4. **Run checks locally**

   At minimum, before opening a PR:

   ```bash
   cargo test
   cargo clippy --all-targets --all-features -- -D warnings
   # Run fuzz targets if your change affects fuzzed components:
   # cargo fuzz run <target>
