# SentinelWX Documentation

This directory contains the design and process documentation for SentinelWX.  
All code and contributions should align with these documents.

## Documents

- `architecture.md`  
  High‑level system architecture, components, data flow, plugin model, and APIs.

- `roadmap.md`  
  Milestones and planned feature sets, from initial core runtime to advanced plugins.

- `rules.md`  
  Project rules and non‑negotiable principles:
  - Safety‑critical and security‑critical mindset.
  - Data sourcing (official vs derived).
  - Privacy, security, and reliability.
  - Rust coding standards (no panics, no unsafe, exhaustive handling).
  - Licensing and scope boundaries.

- `ai_coding_guidelines.md` (or `claude.md`)  
  Strict instructions for AI coding assistants:
  - How to follow architecture and rules.
  - What APIs may be used.
  - Testing and structure requirements.
  - Security and error‑handling constraints.

## How to use these docs

- **Before coding**:  
  Read `rules.md` and `architecture.md` to understand the design and constraints.

- **When planning features**:  
  Check `roadmap.md` to see where a feature fits and whether a milestone already covers it.

- **When using AI for coding**:  
  Ensure prompts and reviews respect `ai_coding_guidelines.md` and `rules.md`.

- **When contributing**:  
  Follow `CONTRIBUTING.md` in the repo root and use these docs as the design source of truth.
