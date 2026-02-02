# SentinelWX Architecture

SentinelWX is a Rust‑based, plugin‑driven, all‑hazards operational console. It ingests official and hyper‑local data feeds, normalizes them into a unified model, and exposes them via a local HTTP API and web UI.

## Top‑level components

- **Core runtime**
  - Rust binary (`sentinelwx`).
  - Manages config, logging, plugin loading, ingest scheduling, in‑memory store, HTTP API, and WebSocket/SSE.
- **Plugins**
  - Dynamically registered data providers, diagnostics, integrations, and vendor modules.
- **Web UI**
  - Browser‑based, responsive, map‑centric front‑end (desktop + Android/iOS).
- **Storage**
  - In‑memory cache for hot data.
  - Optional persistent store (SQLite/Postgres) for history and replay.

## Data flow overview

1. **Ingest**
   - Async workers per source:
     - NWS API (forecasts, gridpoints, alerts, obs). [NWS API docs: https://www.weather.gov/documentation/services-web-api][web:4]
     - SPC ArcGIS services (outlooks, MDs, watches). [SPC MapServer: https://mapservices.weather.noaa.gov/vector/rest/services/outlooks/SPC_wx_outlks/MapServer][web:63][web:68]
     - NHC GIS/KML/JSON (tracks, cones, wind radii, surge). [NHC GIS: https://www.nhc.noaa.gov/gis/][web:7][web:61][web:66]
     - GOES/GLM and satellite imagery (NOAA/NESDIS, NCEI, open data). [web:84][web:85][web:87][web:90][web:96][web:123][web:127]
     - MRMS/QPE. [web:89][web:92][web:95]
     - NOHRSC/SNODAS snow analyses. [web:98][web:101][web:102][web:108]
     - USGS earthquakes (FDSN, GeoJSON feeds). [web:103][web:109][web:112]
     - NIFC fire outlooks and incident data (where accessible). [web:104][web:107][web:110]
     - NCEI/CDO climate/historical data. [web:64][web:116]
     - mPING, Ambient Weather, personal stations, and vendor feeds (e.g., lightning networks, AllisonHouse). [web:56][web:31][web:37][web:133][web:136][web:139][web:119][web:121]
   - Each ingestor:
     - Fetches over HTTPS using async HTTP.
     - Uses ETag/Last‑Modified or content hashing to detect real updates.
     - Maintains adaptive polling intervals based on observed publish times.

2. **Normalize and store**
   - All inputs normalized into a small set of core types:
     - `Layer` metadata
     - `Feature` (geometry + properties)
     - `TimeSeries` (for station/river/model data)
   - Stored in:
     - An in‑memory cache for live operation.
     - Optional DB for history/replay.

3. **Serve**
   - HTTP API (REST + WebSocket/SSE) provides:
     - `/api/layers` – list of layers from all plugins.
     - `/api/layers/{id}` – current data for a layer (GeoJSON/JSON).
     - `/api/features/{id}` – details for a single feature.
     - `/api/alerts` – active and recent alerts.
     - `/api/modes` – global mode definitions and current mode.
     - `/api/homeassistant/*` – simplified endpoints for Home Assistant. [web:59][web:135]
   - WebSockets/SSE push update notifications to clients when data change.

4. **Display**
   - Web UI renders:
     - Map (Leaflet/MapLibre/OpenLayers).
     - Layer toggles grouped by hazard type and source.
     - Time slider and animation controls.
     - Role presets (IC/EM/SC/OM) and global modes (Severe, Winter, Tropical, Hydro, Fire, Quakes, AQ/Smoke, Quiet Monitoring, Training, etc.).
     - Detail panels for clicked features.

## Core modules (Rust crates / modules)

- `sentinelwx-core`
  - Config loading (TOML/YAML).
  - Logging and tracing.
  - Plugin discovery and registration.
  - Central store (in‑memory + optional DB).
  - Scheduler for ingest tasks, including adaptive timing.

- `sentinelwx-api`
  - HTTP server (Axum/Actix).
  - REST routes and WebSocket/SSE endpoints.
  - Authentication/authorization hooks (for future multi‑user/remote use).

- `sentinelwx-model`
  - Core data structures:
    - `Layer`, `LayerSource`, `Geometry`, `Feature`, `TimeSeries`.
    - `AlertRule`, `AlertEvent`.
    - `Mode` (global mode/profile).
  - Serialization (Serde) and versioning.

- `sentinelwx-plugins-*` (examples)
  - `sentinelwx-plugins-nws`
  - `sentinelwx-plugins-spc`
  - `sentinelwx-plugins-nhc`
  - `sentinelwx-plugins-goes_glm`
  - `sentinelwx-plugins-mrms`
  - `sentinelwx-plugins-nohrsc`
  - `sentinelwx-plugins-usgs`
  - `sentinelwx-plugins-nifc`
  - `sentinelwx-plugins-cdo`
  - `sentinelwx-plugins-mping`
  - `sentinelwx-plugins-ambient`
  - `sentinelwx-plugins-lightning-vendor`
  - `sentinelwx-plugins-allisonhouse`
  - `sentinelwx-plugins-hyperlocal`
  - `sentinelwx-plugins-diagnostics` (mesoscale parameters, soundings, cross‑sections)
  - (future) `sentinelwx-plugins-forecast`

Each plugin implements common traits (`DataProvider`, `LayerProvider`, `ToolProvider`) defined in `sentinelwx-core`.

## Plugin API (high‑level)

Traits (conceptual):

- `LayerProvider`
  - `fn layers(&self) -> Vec<LayerMetadata>`
  - `async fn fetch_layer(&self, id: LayerId, time: Option<DateTime>) -> Result<LayerData>`

- `DataProvider`
  - `async fn ingest(&mut self, store: &mut Store, scheduler: &mut Scheduler)`

- `ToolProvider`
  - For things like soundings, cross‑sections, diagnostics, forecast tools.

Plugins are registered either:
- By static list in `Cargo.toml` (monorepo approach), and/or
- By dynamic loading from a `plugins/` directory (future).

## Global modes and profiles

A Mode is a config object that defines:

- Default role/profile (IC/EM/SC/OM, Public Info, Training, etc.).
- Default visible layers.
- Layer styling preferences and z‑order.
- Update intensity per source (fast/normal/slow).
- Alert rules enabled by default.
- UI layout hints (which side panels / tools shown).

Modes are stored in config and can be added/edited without code changes.

## Alert engine

- Alert rules are defined against:
  - Locations/areas (points, polygons).
  - Fields from layers/time‑series (e.g., warning type, CAPE, shear, QPE, obs, lightning density).
- Evaluated on each relevant data update.
- When a rule transitions from false → true, an `AlertEvent` is created, stored, and pushed to clients and integrations.

Integrations:
- Local notifications, sound cues.
- Web UI banners.
- Home Assistant/webhooks for automations. [web:59][web:131][web:135][web:141]

## Privacy and security

- Default deployment: local‑only, no external telemetry.
- Outbound requests only to configured official/vendor endpoints.
- No user tracking; logs focus on system health and data flow.
- Optional auth for multi‑user/remote setups.
