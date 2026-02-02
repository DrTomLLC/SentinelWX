# SentinelWX Freeze Policy and Registry

This file defines the freeze policy and tracks frozen or review‑only files.

## Policy

States:

- `mutable` – default; normal edits allowed.
- `review_only` – AI must not edit; humans edit sparingly with justification.
- `frozen` – no edits except critical bug or security fixes with explicit approval.

Rules:

- Frozen and review_only files:
  - Must not be modified by AI unless the user explicitly overrides.
  - Should be changed by humans only in narrow, well‑justified cases.
- When a file is frozen:
  - Add or update the corresponding entry in the registry below.
  - Add an in‑file marker at the top of the file, e.g.:

    ```rust
    // SENTINELWX-FROZEN: Do not modify except for critical bug/security fixes with human approval.
    ```

- When unfreezing:
  - Update the registry and remove or adjust the in‑file marker.
- Do not freeze large swaths of the codebase prematurely; freeze once modules are stable and well‑tested. [web:183][web:213]

## Registry

| state       | path                          | reason                             | date       | approver |
|------------|-------------------------------|------------------------------------|------------|----------|
| mutable    | sentinelwx/src/main.rs        | initial state                      |            |          |
| mutable    | sentinelwx/src/config.rs      | initial state                      |            |          |
| mutable    | docs/architecture.md          | living document                    |            |          |

(Add entries here as files become `review_only` or `frozen`.)

