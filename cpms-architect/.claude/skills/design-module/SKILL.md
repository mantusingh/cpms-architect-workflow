---
name: design-module
description: >
  Shortcut entry point that starts a scoped C4 workflow for a specific OCPP/OCPI
  module, profile, or feature. Equivalent to running /architect in Mode B or Mode C,
  but lets the user specify the scope upfront. The SAME C4 model applies:
  Context → Container → Component → Data Models → API Design, with approval gates.
  Usage: /design-module <module-name>
  Examples: /design-module smart-charging, /design-module ocpi-cdr,
            /design-module fleet-management, /design-module plug-and-charge
tools:
  - Read
  - Write
  - Bash
---

# /design-module Skill

## Purpose
Start a scoped C4 design workflow for a specific OCPP/OCPI module, profile,
or feature. This is a convenience alias for `/architect` with the scope
pre-specified. The full C4 framework applies — no shortcuts.

---

## Important

This skill uses **the same C4 workflow** as full-system design:
```
Intake → /c4-context → /c4-container → /c4-component → /data-models → /api-design
```
The only difference is **scope** — the C4 diagrams and documents are focused on
the module's boundary rather than the entire CPMS. The rigor, approval gates,
and output quality standards are identical.

---

## Steps

### 1. Identify Module / Feature

Parse the argument after `/design-module`. If ambiguous, ask:
"What exactly are you designing? Please give me the module name and, if applicable,
which OCPP/OCPI specification version you're targeting."

Map to specification:

| Name | Spec | Section |
|------|------|---------|
| `smart-charging` | OCPP 2.0.1 Part 2 | §K |
| `authorization` | OCPP 2.0.1 Part 2 | §9 |
| `local-auth-list` | OCPP 2.0.1 Part 2 | §9.4 |
| `device-management` | OCPP 2.0.1 Part 2 | §8 |
| `reservation` | OCPP 2.0.1 Part 2 | §10 |
| `firmware-management` | OCPP 2.0.1 Part 2 | §F |
| `plug-and-charge` | OCPP 2.0.1 Part 2 + OCPI Token | §M |
| `diagnostics` | OCPP 2.0.1 Part 2 | §G |
| `ocpi-locations` | OCPI 2.2.1 | §7 |
| `ocpi-sessions` | OCPI 2.2.1 | §9 |
| `ocpi-cdr` | OCPI 2.2.1 | §10 |
| `ocpi-tariffs` | OCPI 2.2.1 | §11 |
| `ocpi-tokens` | OCPI 2.2.1 | §12 |
| `ocpi-commands` | OCPI 2.2.1 | §13 |
| `ocpi-charging-profiles` | OCPI 2.2.1 | §14 |

For any module not in this table (e.g., "fleet-management", "reservation-portal"):
treat as a feature design and detect automatically whether it is Mode B (new module
within an existing CPMS concept) or Mode C (incremental feature on approved design).

---

### 2. Check Existing Design

Read `workflow-state.json`.

- If phases are approved in the full-system design: this is **Mode C (Incremental)**.
  Read existing C4 documents. Confirm:
  "I see an existing approved design for [scope]. I'll design [module] as an
  incremental feature using a C4 delta approach. Output will go to
  `output/features/<module-slug>/`."

- If no full-system design exists: this is **Mode B (Scoped Module)**.
  Confirm: "No existing system design found. I'll design [module] as a standalone
  scoped C4 workflow. Output will go to `output/c4-level-*/` with module prefix."

---

### 3. Delegate to /architect Logic

From this point forward, execute the appropriate mode from the `/architect` skill:
- Mode B logic for standalone module design
- Mode C logic for incremental feature addition

This includes:
- All intake questioning (module-specific + general baseline where applicable)
- Dynamic follow-up batches
- "Ready to Design" confirmation before any output
- Impact Assessment (Mode C only)
- Same C4 phases with same approval gates
- Same output quality standards (Mermaid + draw.io, ADRs, spec citations)

The skills invoked after intake are the same: `/c4-context`, `/c4-container`,
`/c4-component`, `/data-models`, `/api-design`.

---

## C4 Scoping for Modules

When running C4 for a module scope, interpret each level appropriately:

**Level 1 — Context (scoped)**
Shows this module's role within the broader CPMS, its external actors (if any),
and the external systems it interacts with directly. Does NOT show the entire CPMS
context — only the module's boundary.

**Level 2 — Container (scoped)**
Shows the services, data stores, and Kafka topics directly involved in this module.
Cross-references existing services from the approved design where applicable.
New services or modifications to existing services are clearly marked.

**Level 3 — Component (scoped)**
Shows internal Go package structure for the primary service(s) implementing this module.
Only produces component diagrams for services that have new or modified components.

**Level 4 — Data Models (scoped)**
New PostgreSQL tables, modified tables (delta DDL only), new Kafka message schemas,
new Redis key patterns. References existing tables by name — does not re-document them.

**API Design (scoped)**
New gRPC methods (proto additions), new GraphQL types/mutations/subscriptions,
new OCPI endpoints. For modified contracts: shows the diff and assesses breaking changes.
