# SentinelWX Rules

Core principles and non‑negotiables for this project.

SentinelWX is treated as **safety‑critical** software. Design and implementation must assume that lives can depend on correct behavior.

---

## 1. Data sourcing

- Use **official, authoritative sources** for all operational data whenever possible:
  - NWS API (forecasts, alerts, gridpoints, obs). [web:4][web:56][web:57][web:62][web:67]
  - SPC products via official NOAA services (outlooks, MDs, watches). [web:63][web:68]
  - NHC GIS/KML/JSON feeds (tracks, cones, wind radii, surge). [web:7][web:61][web:66]
  - NOAA/NESDIS/NCEI satellite and GOES‑R/GLM products. [web:84][web:85][web:87][web:90][web:96][web:120][web:123][web:127]
  - MRMS/QPE and related precipitation/mosaic products. [web:89][web:92][web:95]
  - NOHRSC/SNODAS snow and snowmelt products. [web:98][web:99][web:100][web:101][web:102][web:105][web:108][web:111]
  - NCEI/Climate Data Online for historical/climate data. [web:64][web:116]
  - USGS earthquakes and related feeds. [web:103][web:106][web:109][web:112]
  - NIFC / wildfire services for fire outlooks and incidents. [web:104][web:107][web:110]
  - Ambient Weather, mPING, and similar for hyper‑local data where officially documented. [web:56][web:59][web:133][web:136][web:139]

- Vendor/commercial data (e.g., lightning networks, AllisonHouse, other paid feeds) must:
  - Be implemented as separate plugins.
  - Be clearly tagged and attributed by source in the UI and API. [web:31][web:37][web:119][web:121]

---

## 2. Separation of official vs derived

- **Official products** (e.g., warnings, outlooks, advisories, official analyses) must:
  - Be clearly identified by:
    - Source agency (NWS, SPC, NHC, NOAA, USGS, NIFC, etc.).
    - Product name and type.
    - Issuance and expiration times. [web:4][web:63][web:7][web:98][web:103][web:104][web:107][web:109][web:116]
  - Be displayed without altering their meaning, geometry, or validity windows.

- **Derived products** (indices, composites, forecasts, AI/ML outputs, heuristic metrics such as “mold risk”):
  - Must be visually and textually marked as **“derived / non‑official”**.
  - Must not be misrepresented as official guidance or products.
  - Should, where possible, document which inputs and methods were used.

---

## 3. Privacy and security

SentinelWX must be written as security‑critical software.

- **Least privilege and minimal exposure**
  - Default bind is **localhost only**. Any LAN or remote exposure must be explicitly enabled.
  - Only the minimal required ports and endpoints are exposed.
  - No hidden admin or debug endpoints.

- **No unauthenticated write paths**
  - Any operation that mutates state (rules, incidents, alerts, config, plugin settings, user data) must:
    - Require explicit authentication/authorization when the service is reachable beyond localhost.
    - Be disabled or restricted by default in multi‑user/remote deployments.

- **Secure configuration and secrets**
  - Do **not** hard‑code secrets, API keys, or passwords.
  - Secrets are loaded from environment variables or dedicated secret stores, not committed to the repo. [web:59][web:133][web:136][web:139]
  - Configuration is strictly validated:
    - Types, ranges, and formats checked.
    - Invalid config causes clear, early failure with explicit error messages.

- **Transport security**
  - When accessed beyond localhost, SentinelWX must be behind TLS (reverse proxy or built‑in HTTPS).
  - Credentials and sensitive data must not be transmitted in plaintext over untrusted networks.

- **Input validation and robustness**
  - Treat **all** external input as untrusted:
    - API responses, configs, user input, web requests, plugin inputs.
  - Validate lengths, ranges, formats, and invariants before use.
  - Reject or sanitize malformed data and handle with explicit errors; never panic.

- **No code injection / unsafe eval**
  - No dynamic code execution from untrusted input:
    - No `eval`‑style behavior.
    - No shelling out with unsanitized user input.
  - Any optional scripting hooks must:
    - Be disabled by default.
    - Be clearly documented and sandboxed.

- **Dependency hygiene**
  - Prefer well‑maintained crates with good security records.
  - Keep dependencies reasonably up to date.
  - Run security tools (e.g., `cargo audit`) periodically and address findings. [web:151]

- **Logging and PII**
  - Logs must not contain:
    - Secrets, API keys, tokens, passwords.
    - Sensitive personal data beyond what is operationally necessary.
  - Logs should contain enough context for debugging without leaking private information.

- **Defense in depth**
  - Assume bugs and misconfigurations will occur:
    - Layer validation, authorization, rate limiting, and error handling.
    - Design so a single bug is unlikely to lead to catastrophic compromise.

- **Security review**
  - Any change that touches:
    - Authentication/authorization,
    - Network exposure,
    - Secret handling,
    - Plugin loading or dynamic behavior,
  requires explicit human review with a security mindset.

---

## 4. Reliability and resilience

- The system must **fail gracefully**:
  - If a data source is down or returns invalid data:
    - Mark its status (OK / degraded / down).
    - Use cached data where safe and clearly show that it is stale.
    - Do not crash the core process.

- Ingest and alert evaluation are isolated:
  - Each plugin or ingest worker must not be able to crash others or the core.
  - Use robust error handling around all plugin boundaries.

- Use adaptive backoff and rate limiting:
  - Avoid hammering upstream APIs.
  - Escalate backoff on repeated failures, with clear logging and status indicators.

---

## 5. UX under stress

- **Map‑first and role‑driven**
  - The main view is always the map with relevant overlays.
  - Role presets (IC, EM, SC, OM, Public Info, etc.) and global modes (Severe, Winter, Tropical, Hydro, Fire, Quakes, AQ/Smoke, Quiet, Training) must be easily switchable.

- **Two‑click rule for critical actions**
  - Changing global mode, jumping to an Area of Operations (AO), and viewing active alerts should be 1–2 clicks/taps.

- **No modal traps**
  - Avoid full‑screen blocking dialogs for operational information.
  - Use side/bottom panels and non‑blocking notifications; the map must remain visible wherever possible.

- **Keyboard shortcuts**
  - Provide keyboard shortcuts on desktop for frequent ops actions.

---

## 6. Alerts

- **Alert rules**
  - Are explicitly defined, visible, and configurable.
  - Operate on:
    - Official products (warnings, outlooks, advisories, QPE, etc.). [web:4][web:63][web:7][web:89][web:92][web:95][web:98][web:101][web:102][web:104][web:107][web:108][web:109][web:116][web:117][web:121]
    - Hyper‑local data (stations, mPING, Ambient). [web:56][web:133][web:136][web:139]
    - Vendor data where explicitly enabled (e.g., lightning strokes with polarity and kA). [web:119][web:121]

- **Alert behavior**
  - Alerts must include:
    - What triggered (rule name and condition).
    - Where (location or area).
    - When (timestamp).
    - Why (key values/fields that crossed thresholds).
  - Alerts must be:
    - Throttled (per‑rule cooldowns).
    - Deduplicated (group related updates).
    - Tiered (Info / Heads‑Up / Immediate) to avoid alert fatigue.

- **Integrations**
  - Alerts may be delivered to:
    - Local UI (banners, sounds, focus).
    - System/OS notifications.
    - Home Assistant/webhooks for automations. [web:59][web:131][web:135][web:141]
  - Integrations must not compromise security (no unauthenticated arbitrary callbacks).

---

## 7. Licensing and contributions

- SentinelWX is **dual‑licensed**:
  - AGPL‑3.0 for open‑source use and contributions.
  - A proprietary commercial license for organizations that need closed modifications or distribution. [web:145][web:149][web:150][web:152][web:156]

- Contributions:
  - Are accepted under AGPL‑3.0 terms and may be included in commercial builds by the project owner (standard dual‑licensing model).
  - Must not incorporate code from incompatible licenses.

---

## 8. Scope boundaries

- SentinelWX is primarily a **monitoring, alerting, and decision‑support** system.
- Any custom forecasting/AI/ML components:
  - Must be implemented as separate plugins.
  - Must be clearly marked as “derived / non‑official”.
  - Must never override or hide official forecasts, warnings, or directives.

---

## 9. Coding standards (Rust)

These are non‑negotiable technical rules:

- **No panics in production**
  - No use of `unwrap()`, `expect()`, `panic!()`, or `assert!()` in production code paths.
  - All fallible operations must return and handle `Result` / `Option` explicitly.

- **Exhaustive error handling**
  - Handle all match arms; avoid `_` except with clear, explicit behavior.
  - No silent error ignoring (e.g., `let _ = …;`) without a documented, logged rationale.

- **No `unsafe`**
  - No `unsafe` blocks unless absolutely unavoidable.
  - Any `unsafe` must:
    - Be justified in comments.
    - Be reviewed carefully.
    - Be isolated and wrapped in safe abstractions.

- **File size limits**
  - Target < 250 LOC per source file; hard maximum 400 LOC excluding comments/whitespace.
  - Refactor and split modules before exceeding these limits.

- **Rust version / edition**
  - Code must compile with:
    - Rust **1.93**,
    - Edition **2024**.
  - No nightly‑only features.

- **Testing**
  - All logic must be covered by tests in a separate `tests/` directory:
    - No `#[cfg(test)]` inline unit tests in production source files.
    - Tests must include normal, edge, and error cases.
  - When behavior is unclear:
    - Write or extend tests first to encode the intended behavior.
    - Adjust implementation to satisfy the tests and these rules.

- **Clarity over cleverness**
  - Prefer explicit, straightforward code over clever or heavily abstracted patterns.
  - Use clear naming and Rustdoc comments for public APIs.

---

## 10. Security and trust (summary)

- Design as if:
  - The system will be deployed in exposed environments.
  - A failure could cost lives.
- Adhere to:
  - Least privilege.
  - Strong input validation.
  - No dynamic code exec from untrusted input.
  - Secure configuration and logging.
- All security‑sensitive changes require careful human review.

---

## 11. When in doubt

- If there is any uncertainty about requirements or behavior:
  - Consult `architecture.md`, this `rules.md`, and `ai_coding_guidelines.md`.
  - Write or update tests to define expected behavior.
  - Ask for human review and clarification before merging changes.

These rules exist to ensure SentinelWX remains trustworthy, predictable, and suitable for life‑critical operational use.
