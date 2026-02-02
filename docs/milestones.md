# SentinelWX Milestones

This file connects the high‑level roadmap (`docs/roadmap.md`) with concrete progress.  
Each milestone lists its major goals and points to tasks in `docs/tasks.md`.

Milestones should be updated as work is completed or re‑scoped.

---

## Milestone 0 – Repo, docs, and skeleton

Goal: Minimal, compilable Rust workspace with documentation and health check skeleton.

- Status: [ ] Not started / [ ] In progress / [ ] Complete
- Related roadmap section: “Milestone 0 – Repository, docs, and skeleton” in `docs/roadmap.md`.
- Tasks:
  - See `docs/tasks.md` under “Milestone 0”.

---

## Milestone 1 – Core runtime and HTTP API skeleton

Goal: Config, logging, in‑memory store, versioned HTTP API with basic endpoints.

- Status: [ ] Not started / [ ] In progress / [ ] Complete
- Related roadmap section: “Milestone 1 – Core runtime and HTTP API skeleton”.
- Tasks:
  - See `docs/tasks.md` under “Milestone 1”.

---

## Milestone 2 – Official feed ingestion (MVP core feeds)

Goal: NWS, SPC, NHC, and radar/MRMS ingestion with normalized layers.

- Status: [ ] Not started / [ ] In progress / [ ] Complete
- Related roadmap section: “Milestone 2 – Official feed ingestion (MVP core feeds)”.
- Tasks:
  - See `docs/tasks.md` under “Milestone 2”.

---

## Milestone 3 – Map UI, roles, and basic modes

...

(Repeat this pattern for Milestones 3–8 as described in `docs/roadmap.md`.)

For each milestone:

- Only mark “Status: Complete” when:
  - All associated tasks in `docs/tasks.md` are checked.
  - Tests, clippy, and relevant fuzzers are clean.
  - Any planned freezes for stable modules have been applied (if desired).

