# Milestones – status and task linkage

This document provides a thin, human‑readable bridge between the high‑level plan in `docs/roadmap.md` and the concrete work items in `docs/tasks.md`.  
It is intentionally short: goals, status, and a pointer to the corresponding task section.

For detailed milestone scope and design, see `docs/roadmap.md`.  
For specific tasks and acceptance criteria, see `docs/tasks.md`.

## Status legend

Use the following status values:

- `Not started` – Milestone is defined and approved but no tasks have been meaningfully executed.
- `In progress` – Work has begun; at least one related task is underway or complete, but the milestone is not yet fully satisfied.
- `Complete` – All required tasks for this milestone are complete, validated, and merged, with checks passing (tests, clippy, fuzz where applicable).

Only human maintainers should change milestone status values.

---

## Milestone M0 – Bootstrapping and control layer

- Goal: Establish a minimal Rust skeleton, core documentation, and strict AI control/freeze model with tests‑first behavior, but no real weather ingestion or UI yet.
- Status: Not started
- Tasks: See milestone M0 section in `docs/tasks.md`.

Notes for maintainers:

- Consider M0 “Complete” only when:
  - Core control docs are aligned and stable:
    - `.aicode`
    - `docs/rules.md`
    - `docs/freeze.md`
    - `docs/ai_coding_guidelines.md`
    - `docs/README.md`
  - The basic test harness structure exists and is in active use.
  - Root `README.md` and `CONTRIBUTING.md` reflect the tests + clippy + fuzz philosophy and freeze model.
  - All relevant tasks under M0 in `docs/tasks.md` are marked `[x]` and their criteria have been satisfied.

---

## Milestone M1 – Core runtime skeleton

- Goal: Build the minimal runtime skeleton (configuration loading, logging, and entrypoint) with safety‑critical defaults, but without real external weather sources.
- Status: Not started
- Tasks: See milestone M1 section in `docs/tasks.md`.

Notes for maintainers:

- Mark M1 “In progress” when:
  - At least one M1 task is actively being worked on and has a clear owner (human or AI).
- Mark M1 “Complete” only when:
  - Configuration and logging scaffolding are implemented as described in `docs/roadmap.md` and `docs/architecture.md`.
  - Startup paths are covered by tests and do not panic under normal conditions.
  - All M1 tasks in `docs/tasks.md` are `[x]`, with tests, clippy, and relevant fuzz targets passing.

---

## Milestone M2 – Data ingestion scaffolding

- Goal: Define initial internal data models and placeholder ingestion pathways for official weather and hazard data, without full external connectivity.
- Status: Not started
- Tasks: See milestone M2 section in `docs/tasks.md`.

Notes for maintainers:

- Treat M2 as “In progress” once initial model types or ingestion scaffolds are being implemented according to `docs/roadmap.md`.
- Treat M2 as “Complete” only when:
  - Core internal models exist and are covered by tests.
  - Any proto‑ingestion logic respects safety and validation rules from `docs/rules.md`.
  - All M2 tasks in `docs/tasks.md` are `[x]`, with tests, clippy, and relevant fuzz targets passing.

---

## Milestones M3–M8 – Future phases

The detailed goals and sequencing for milestones M3 through M8 are defined in `docs/roadmap.md`.  
Tasks for these milestones will be added to `docs/tasks.md` as implementation approaches each phase.

- M3 – Initial ingest and normalization pipeline
  - Goal: Implement early ingest/normalize paths for selected official data sources, with strict validation and test coverage.
  - Status: Not started
  - Tasks: See milestone M3 section in `docs/tasks.md` (to be expanded by maintainers).

- M4 – Core mapping and visualization
  - Goal: Provide map‑centric visualization of normalized data with safety‑critical defaults for display and interaction.
  - Status: Not started
  - Tasks: See milestone M4 section in `docs/tasks.md` (to be defined).

- M5 – Alerting and decision support
  - Goal: Add alerting, decision support, and workflow hooks tailored to incident command and serious weather operations.
  - Status: Not started
  - Tasks: See milestone M5 section in `docs/tasks.md` (to be defined).

- M6 – Plugin and extension ecosystem
  - Goal: Introduce a controlled plugin/extension model that allows new data sources and outputs while preserving safety guarantees.
  - Status: Not started
  - Tasks: See milestone M6 section in `docs/tasks.md` (to be defined).

- M7 – Hardening, security, and reliability
  - Goal: Harden SentinelWX for real operational use: security, performance, redundancy, and resilience improvements.
  - Status: Not started
  - Tasks: See milestone M7 section in `docs/tasks.md` (to be defined).

- M8 – UX refinement and operational polish
  - Goal: Refine UX, documentation, and operational polish for high‑stress workflows (IC, EM, chase ops), without compromising safety.
  - Status: Not started
  - Tasks: See milestone M8 section in `docs/tasks.md` (to be defined).

---

## Updating this file (humans only)

- Only human maintainers should:
  - Change milestone status.
  - Adjust milestone goal wording (if roadmap scope truly changes).
  - Add or remove milestones.
- When you update a milestone’s status:
  - Ensure the corresponding tasks in `docs/tasks.md` reflect reality (e.g., completed tasks are `[x]`).
  - Confirm that test, clippy, and fuzz conditions for relevant tasks are still being enforced.
- AI tools must not:
  - Change milestone status.
  - Rewrite goals.
  - Add or remove milestones.
  - They may only **report** current status and suggest updates when explicitly asked; humans decide what to apply.
