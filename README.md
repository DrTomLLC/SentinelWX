# SentinelWX

SentinelWX is a safety‑critical weather situational awareness and operations tool.  
It ingests official weather and hazard data, normalizes it into a common model, and exposes it through a map‑centric UI and APIs for incident command, emergency management, and serious weather users. [web:209]

## Status

This project is in early development.  
APIs, plugins, and UI are unstable and may change without notice.

## Development philosophy

SentinelWX uses a **tests‑first** approach wherever possible:

- New behavior should be driven by tests (unit, integration, or fuzz tests) before or alongside implementation. [web:206]  
- Code must compile cleanly, pass all tests, and pass linting (`cargo clippy`) with warnings treated as errors. [web:208][web:211]  
- For critical parsing and boundary logic, fuzzing is strongly encouraged with tools like `cargo fuzz` or similar fuzz harnesses. [web:209]

Automation and AI tools (Claude Code, Antigravity, etc.) are expected to:

- Pick tasks from `docs/tasks.md`.
- Write or update tests first.
- Run `cargo test`, `cargo clippy`, and any configured fuzz targets.
- Use the output to iteratively correct issues until the tree is clean.

## Features (planned)

- Core Rust backend with strict safety and reliability rules.
- Plugin system for ingesting official feeds (NWS, SPC, NHC, radar, etc.).
- Map UI with layers, time controls, and role/mode presets.
- Alert rules engine for AO polygons and hyper‑local data.
- Optional integrations (Home Assistant, vendor lightning, mesonets, training/replay).

For detailed milestones, see `docs/roadmap.md` and `docs/milestones.md`.

## Getting started

### Prerequisites

- Rust (stable toolchain).
- A supported OS (Linux, macOS, or Windows).
- Access to the public internet for official data sources.

### Build and run

```bash
git clone https://github.com/<you>/sentinelwx.git
cd sentinelwx
cargo run
