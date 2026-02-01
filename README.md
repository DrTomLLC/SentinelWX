# SentinelWX
# SentinelWX

All‑hazards, real‑time operational console for IC, EM, storm chasing, and operational meteorology, powered by official NOAA, USGS, and hyper‑local data.

## Project status

Early design / pre‑alpha. Core architecture and requirements are being defined before implementation.

## Goals

- Single, professional console for:
  - Incident Command (IC)
  - Emergency Management (EM)
  - Storm Chasing (SC)
  - Operational Meteorology (OM)
- Fuse official, authoritative feeds (NWS, SPC, NHC, NOAA, USGS, etc.) with hyper‑local data (personal stations, mPING, Ambient Weather, vendor feeds).
- Near real‑time ingest and display, with per‑feed adaptive scheduling and push updates.
- Clean, map‑first UI with role‑based global modes (Severe, Winter, Tropical, Hydro, Fire, Quakes, AQ/Smoke, OM, Chase, etc.).
- Hyper‑local, rule‑based alerting for specific locations and areas.
- Plugin‑driven architecture for data sources, diagnostics, and vendors.
- Runs locally with maximum privacy; no third‑party tracking.

## High‑level architecture

- **Core runtime (Rust)**
  - Async ingest workers per source (NWS, SPC, NHC, GOES/GLM, MRMS/QPE, NOHRSC/SNODAS, USGS quakes, NIFC fire, mPING, Ambient, optional vendors).
  - Central store + cache (time‑series + geospatial).
  - Plugin system for data providers, diagnostics, and vendor integrations.
  - HTTP API (REST + WebSocket/SSE) serving a local web UI and integrations (Home Assistant, etc.).

- **Plugins**
  - Official feeds: NWS, SPC, NHC, GOES/GLM, MRMS/QPE, NOHRSC, USGS, NIFC, NCEI/CDO, etc.
  - Hyper‑local: Ambient Weather, personal stations, mPING, local incidents/resources.
  - Vendor: lightning networks, AllisonHouse, and other paid sources (optional).
  - Diagnostics/forecasting: mesoscale parameters, soundings, cross‑sections, model browser (later).

- **Front‑end**
  - Browser‑based, responsive map UI (desktop + Android/iOS).
  - Role presets (IC/EM/SC/OM) and global modes (Severe, Winter, Tropical, Hydro, Fire, etc.).
  - Layer manager, time slider, detail panels, alert indicators.

## Licensing

SentinelWX is dual‑licensed:

- **Open‑source:** AGPL‑3.0, for community use and contributions.  
- **Commercial:** A separate proprietary license is available for organizations that wish to use or modify SentinelWX without the obligations of the AGPL‑3.0.

See `LICENSE` and `LICENSE-COMMERCIAL` for details.
