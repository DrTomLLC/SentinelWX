# SentinelWX Roadmap

This roadmap is high‑level and safety‑critical.  
Each milestone should be implemented in small, well‑tested steps, with issues and PRs tied back to these items.

All work must respect:

- `docs/rules.md`
- `docs/architecture.md`
- `docs/ai_coding_guidelines.md`

---

## Milestone 0 – Repository, docs, and skeleton

**Goal:** A clean, well‑documented repo with a minimal, compilable Rust skeleton and no real functionality.

- [x] Create GitHub repo and initial description.
- [ ] Create `docs/` and add:
  - [ ] `docs/README.md`
  - [ ] `docs/architecture.md`
  - [ ] `docs/roadmap.md`
  - [ ] `docs/rules.md`
  - [ ] `docs/ai_coding_guidelines.md`
- [ ] Add project meta files:
  - [ ] `CONTRIBUTING.md`
  - [ ] `CODE_OF_CONDUCT.md` (can use GitHub template)
  - [ ] `LICENSE` (AGPL‑3.0)
  - [ ] `LICENSE-COMMERCIAL` (dual‑license notice)
- [ ] Set up Rust workspace:
  - [ ] `Cargo.toml` with workspace definition and `sentinelwx` binary crate.
  - [ ] `src/main.rs` with:
    - Minimal logging/tracing.
    - HTTP server skeleton with `/health` returning `200 OK`.
- [ ] Add `tests/README.md` and at least one basic integration test hitting `/health`.

---

## Milestone 1 – Core runtime and HTTP API skeleton

**Goal:** Robust core process with config, logging, a basic store, and a versioned HTTP API surface (no real data yet).

- [ ] Config and logging
  - [ ] Implement structured config loading (TOML/YAML).
  - [ ] Implement structured logging with `tracing`, including log levels and components.
  - [ ] Validate config on startup; fail fast on invalid config.

- [ ] Core store and types
  - [ ] Implement basic in‑memory store interfaces for:
    - Layers metadata.
    - Features.
    - Alerts.
    - Source status/health.
  - [ ] Define core model types in a dedicated crate/module (`sentinelwx-model`).

- [ ] HTTP API skeleton (no real data yet)
  - [ ] Implement:
    - `GET /health`
    - `GET /api/layers` (returns empty list plus version info)
    - `GET /api/modes` (returns stub modes)
    - `GET /api/status` (returns per‑source status placeholders)
  - [ ] Wire WebSocket/SSE endpoint placeholder for future push updates.

- [ ] Tests
  - [ ] Integration tests for `/health`, `/api/layers`, `/api/modes`, `/api/status`.
  - [ ] Unit tests for config parsing, default modes, and error handling.

---

## Milestone 2 – Official feed ingestion (MVP core feeds)

**Goal:** Ingest a minimal, high‑value set of official feeds and expose them through the API and store, with adaptive scheduling.

### 2.1 NWS core plugin

- [ ] Implement `sentinelwx-plugins-nws`:
  - [ ] Fetch NWS alerts (watches/warnings/advisories) via NWS API. [web:4][web:56][web:57][web:62][web:67]
  - [ ] Normalize alerts into `Feature` objects with geometry, type, and times.
  - [ ] Expose alert layers via `/api/layers` and `/api/layers/{id}`.

- [ ] Add adaptive scheduling:
  - [ ] Poll alerts endpoint frequently (e.g., 15–30s) with ETag/Last‑Modified.
  - [ ] Back off on repeated failures; mark source status.

- [ ] Tests:
  - [ ] Parsing of example NWS alert JSON (valid, missing fields, malformed).
  - [ ] Scheduler behavior (update detection, backoff).

### 2.2 SPC plugin (outlooks and watches)

- [ ] Implement `sentinelwx-plugins-spc`:
  - [ ] Fetch SPC outlook polygons from official MapServer (Day 1–3 at least). [web:63][web:68]
  - [ ] Fetch SPC watch polygons (directly or via trusted intermediates like IEM where appropriate). [web:65]
  - [ ] Normalize to `Layer` + `Feature` with metadata and validity.

- [ ] Tests:
  - [ ] Parsing SPC JSON/GeoJSON samples (valid, edge, malformed).
  - [ ] Handling of multiple categories (Marginal/SLGT/ENH/etc.).

### 2.3 NHC plugin (tropical storms)

- [ ] Implement `sentinelwx-plugins-nhc`:
  - [ ] Use NHC active storm feeds (GIS/KML/JSON). [web:7][web:61][web:66]
  - [ ] Extract storm tracks, forecast cones, wind radii, key metadata.
  - [ ] Expose storm and cone layers.

- [ ] Tests:
  - [ ] Parsing KML/JSON for storms at different stages.
  - [ ] Handling of storms starting/ending, adv cycle changes.

### 2.4 Radar/MRMS base

- [ ] Implement `sentinelwx-plugins-radar` or `mrms`:
  - [ ] Integrate with MRMS radar reflectivity or NWS radar mosaic via official map/image services. [web:89][web:92][web:95]
  - [ ] Represent radar as tile URLs or equivalent, not as huge raw arrays in JSON.

- [ ] Tests:
  - [ ] URL construction, time selection, error handling for unavailable tiles.

---

## Milestone 3 – Map UI, roles, and basic modes

**Goal:** Usable web UI with map, layers, time slider, and basic role/mode switching.

- [ ] Web UI base
  - [ ] Set up front‑end (vanilla JS or minimal framework) with:
    - Map component (Leaflet/MapLibre/OpenLayers).
    - Layer overlay handling for:
      - NWS alerts.
      - SPC outlooks & watches.
      - NHC cones/tracks.
      - Radar tiles.
  - [ ] Implement a horizontal time slider for radar/sat loops (simple: frames and play/pause).

- [ ] Roles and global modes (initial set)
  - [ ] Role presets:
    - IC
    - EM
    - Storm Chasing (SC)
    - Operational Meteorology (OM)
  - [ ] Global modes:
    - Quiet Monitoring
    - Severe Convective Ops
    - Tropical / Cyclone Ops
  - [ ] Modes define:
    - Which layers are visible.
    - Basic update intensity (still using same scheduler underneath but with flags for future tuning).

- [ ] Detail panel
  - [ ] Clicking a feature (e.g., warning, outlook area, storm) opens a side/bottom panel with:
    - Official text (NWS alert, SPC MD/outlook summary, NHC advisory links). [web:4][web:63][web:7]
    - Source and timestamps.

- [ ] Tests:
  - [ ] API integration tests verifying modes and layers endpoints.
  - [ ] Front‑end tests (where feasible) for basic layer toggling and mode switching.

---

## Milestone 4 – Alerts and hyper‑local integration (first pass)

**Goal:** Implement rule‑based alerts using official data, and integrate key hyper‑local sources.

### 4.1 Alert engine (phase 1)

- [ ] Define `AlertRule` and `AlertEvent` schemas.
- [ ] Implement evaluation engine:
  - [ ] Supports simple rules like:
    - “Any Tornado Warning intersects AO polygon.”
    - “Any Severe Thunderstorm Warning within X km of point.”
  - [ ] Runs when new alerts are ingested.
- [ ] Integrate with API:
  - [ ] `GET /api/alerts` for active/recent alerts.
- [ ] UI:
  - [ ] Show alert banners and list.
  - [ ] Clicking an alert zooms to area and shows details.

- [ ] Tests:
  - [ ] Rule evaluation for intersects / distance.
  - [ ] Alert creation and cooldown/no‑spam behavior.

### 4.2 Hyper‑local: mPING and Ambient Weather

- [ ] `sentinelwx-plugins-mping`:
  - [ ] Fetch or receive mPING reports (as available). [web:56]
  - [ ] Normalize as point features with type and intensity.
- [ ] `sentinelwx-plugins-ambient`:
  - [ ] Ingest data from Ambient Weather API using documented endpoints. [web:133][web:136][web:139]
  - [ ] Normalize as station observations (time‑series + last value).

- [ ] UI:
  - [ ] Layers for mPING and Ambient stations.
  - [ ] Detail panel for station/report.

- [ ] Tests:
  - [ ] Parsing of mPING and Ambient responses.
  - [ ] Edge cases (offline stations, stale data).

### 4.3 Home Assistant integration (basic)

- [ ] Add `/api/homeassistant/*` endpoints:
  - [ ] Current conditions at a configured point.
  - [ ] Current active alerts for AO.
- [ ] Provide example HA config in docs.

- [ ] Tests:
  - [ ] HA endpoints return stable, documented JSON shapes.

---

## Milestone 5 – Additional hazards, diagnostics, and improved modes

**Goal:** Extend coverage to winter, floods, fire, quakes, AQ/smoke; add first mesoscale diagnostics.

### 5.1 Winter and hydrology

- [ ] Integrate NOHRSC/SNODAS snow analyses. [web:98][web:101][web:102][web:108][web:111]
- [ ] Add river/stream gauge data where available (NWS/USGS).
- [ ] Add MRMS/QPE accumulations and relevant hydro fields. [web:89][web:92][web:95]
- [ ] Create “Winter Storm Ops” and “Hydro / Flood Ops” modes.

- [ ] Tests:
  - [ ] Parsing snow/SWE products.
  - [ ] QPE handling and thresholds for rules.

### 5.2 Fire and air quality

- [ ] Integrate NIFC outlooks and significant fire potential. [web:104][web:107][web:110]
- [ ] Integrate active fire perimeters/incidents where accessible. [web:107]
- [ ] Add at least one AQ/smoke data source where publicly available. [web:116][web:118][web:121]
- [ ] Create “Fire Weather / Smoke” and “Air Quality / Health” modes.

- [ ] Tests:
  - [ ] Parsing fire and AQ layers.
  - [ ] Rules using fire potential + lightning + wind.

### 5.3 Earthquakes and multi‑hazard

- [ ] Integrate USGS earthquake feeds (GeoJSON). [web:103][web:106][web:109][web:112]
- [ ] Create “Earthquake / Multi‑Hazard” mode.

- [ ] Tests:
  - [ ] Parsing quakes.
  - [ ] Alert rules by magnitude / distance.

### 5.4 Basic mesoscale diagnostics

- [ ] Implement first diagnostic plugin:
  - [ ] Simple derived fields (e.g., CAPE thresholds, shear composites) from available gridded data. [web:4][web:87][web:90][web:96]
  - [ ] Strictly labeled as derived / non‑official.
- [ ] Add layers and legend entries for these diagnostics.

- [ ] Tests:
  - [ ] Correct computation for synthetic inputs.
  - [ ] Handling of missing or partial grids.

---

## Milestone 6 – Lightning and vendor plugins

**Goal:** Robust lightning layers and optional per‑stroke vendor integration.

### 6.1 Free/official lightning

- [ ] Integrate GLM‑based products (flash extent density, area, energy). [web:117][web:120][web:123][web:124][web:127]
- [ ] Integrate NOAA lightning strike density (where available). [web:118][web:121]
- [ ] Add lightning density layers and profiles using these.

- [ ] Tests:
  - [ ] Parsing GLM/grid products.
  - [ ] Rendering density into appropriate layers.

### 6.2 Vendor lightning (optional plugin)

- [ ] Implement `sentinelwx-plugins-lightning-vendor`:
  - [ ] Consume stroke‑level API where configured (type, polarity, kA, vendor metrics). [web:119][web:121]
  - [ ] Expose point layers with filterable attributes (CG/IC, +/‑, kA).
- [ ] UI:
  - [ ] Enable advanced lightning view when plugin active.
  - [ ] Clearly label vendor source.

- [ ] Tests:
  - [ ] Parsing vendor responses (success, error).
  - [ ] Filter and styling logic.

### 6.3 AllisonHouse and similar vendors

- [ ] Implement `sentinelwx-plugins-allisonhouse`:
  - [ ] Consume supported data types (radar, mesonet, etc.) where allowed. [web:31][web:37]
  - [ ] Expose as optional layers, clearly labeled.

- [ ] Tests:
  - [ ] API integration behavior under rate limits, errors, and partial data.

---

## Milestone 7 – Replay, training, exports, and ops features

**Goal:** Turn SentinelWX into a full ops tool with replay, training, and product generation.

- [ ] Persistent storage:
  - [ ] Optional DB to record selected layers and alerts over time.
- [ ] Replay mode:
  - [ ] Time slider that replays past events.
  - [ ] Sync radar, sat, hazards, alerts, incidents.
- [ ] Training mode:
  - [ ] Predefined training scenarios using recorded data.
- [ ] Export:
  - [ ] Static map snapshots (PNG/SVG).
  - [ ] Simple animation clips for loops.
- [ ] Ops log:
  - [ ] Append key events (alerts, incidents, mode changes) to a structured log.

- [ ] Tests:
  - [ ] Storage read/write and schema migrations.
  - [ ] Replay correctness for a known recorded case.

---

## Milestone 8 – Forecasting / diagnostics plugin (future)

**Goal:** Add a separate plugin for advanced diagnostics and optional forecasting, clearly separated from official products.

- [ ] Define plugin API for:
  - Diagnostics (soundings, cross‑sections, derived composites).
  - Optional forecast tools (e.g., custom outlook proposals) clearly labeled as non‑official.

- [ ] Implement:
  - [ ] Sounding generator from model/obs grids.
  - [ ] Cross‑section tools.
  - [ ] Additional mesoscale parameters.

- [ ] UI:
  - [ ] “Forecast Tools” panel, visually distinct from official products.
- [ ] Tests:
  - [ ] Diagnostics on synthetic fields.
  - [ ] Edge cases and failure behavior.

---

## Ongoing work

- **Performance and scalability**
  - Optimize ingest, caching, and rendering.
  - Monitor memory and CPU usage under load.

- **Security hardening**
  - Regular dependency and security audits. [web:145][web:149][web:150][web:152][web:156]
  - Review of network exposure, auth, and plugin loading.

- **UX and accessibility**
  - Refine UI for clarity under stress.
  - Improve keyboard navigation and screen‑reader support.

- **Documentation**
  - Keep `docs/architecture.md`, `docs/rules.md`, and this roadmap current.
  - Add scenario‑based guides (IC severe ops, EM winter storm, chase day, etc.).

Every milestone must maintain:

- No panics or unchecked errors.
- No unsafe (without explicit, reviewed exceptions).
- Extensive tests in `tests/`.
- Strict adherence to `docs/rules.md` and `docs/ai_coding_guidelines.md`.
