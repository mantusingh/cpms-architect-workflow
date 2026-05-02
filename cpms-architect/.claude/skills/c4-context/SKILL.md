---
name: c4-context
description: >
  Generate the C4 Level 1 System Context diagram and document for the CPMS.
  Shows the system boundary, all human actors, and all external systems with
  their communication protocols. Requires intake phase to be approved.
  Produces Markdown with embedded Mermaid diagram AND a draw.io XML file.
tools:
  - Read
  - Write
  - Bash
---

# /c4-context Skill

## Purpose
Produce the C4 Level 1 System Context architecture document.

---

## Pre-flight Check

1. Read `workflow-state.json`.
2. Read `docs/learnings/architectural-patterns.md` — surface any relevant learnings
   for the System Context phase before starting.
3. Verify `phases.intake.status == "approved"`. If not: tell the human to complete
   intake first (`/architect`) and then run `/approve intake`. STOP.
4. Verify `phases.c4_level_1_context.status` is `"pending"` or `"in_progress"`.
   If `"approved"`: tell the human this phase is done and suggest `/c4-container`.
   If `"locked"`: same message as step 3.
5. Set `phases.c4_level_1_context.status = "in_progress"` and `started_at = <now>`.
   Save `workflow-state.json`.

---

## Output Files

### Primary: `output/c4-level-1-context/system-context.md`

#### Document Header
```markdown
# C4 Level 1 — System Context: CPMS Platform

**Date**: YYYY-MM-DD
**Phase**: C4 Level 1 — System Context
**Based on**:
- Requirements: [list files from workflow-state.json phases.intake.requirements_files_read]
- Reference docs: [list from phases.intake.reference_docs_available]
- OCPI versions: [from decisions.ocpi_versions]
- OCPP versions: [from decisions.ocpp_versions]

## Summary
[One paragraph: what this document covers, what the CPMS is at the highest level,
who should read this document (architects, product owners, external partners)]
```

#### Decisions Made
Bulleted list of every system boundary and protocol decision made in this phase.

#### Actors Table
List EVERY human actor identified from requirements + clarifying answers.

| Actor | Type | Description | Interacts With CPMS Via |
|-------|------|-------------|-------------------------|
| EV Driver | Person | Starts/stops charging sessions | Mobile app (HTTPS/GraphQL), RFID |
| CPO Operator | Person | Manages charge points and network | Operator Dashboard (HTTPS/GraphQL) |
| Fleet Manager | Person | Manages fleet charging (if in scope) | Fleet Portal (HTTPS/GraphQL) |
| Network Admin | Person | Monitors network health | NOC Dashboard (HTTPS/GraphQL) |
| System Admin | Person | Platform configuration | Admin API (HTTPS/GraphQL) |

Only include actors that are confirmed in scope from Section B answers.

#### External Systems Table
List EVERY external system the CPMS integrates with.

| System | Owner | Protocol | OCPP/OCPI Version | Purpose |
|--------|-------|----------|-------------------|---------|
| OCPP Charge Points | CPOs | OCPP over WSS | 1.6J + 2.0.1 | EV charging hardware |
| OCPI eMSP Partners | eMSPs | REST/HTTPS | OCPI 2.1.1 / 2.2.1 | Roaming interoperability |
| Payment Gateway | [from decisions] | REST/HTTPS | - | Payment authorisation/capture |
| Identity Provider | [from decisions] | OIDC/SAML | - | Driver/operator authentication |
| Push Notification Service | APNs/FCM | HTTPS | - | Driver mobile app notifications |
| Email/SMS Provider | [from decisions] | HTTPS | - | Transactional notifications |
| Grid Operator API | DSO | REST/HTTPS | - | Smart charging signals (if in scope) |
| Monitoring/Observability | AWS/Grafana | HTTPS | - | Metrics, logs, alerts |

Only include systems confirmed in scope from requirements + clarifying answers.

#### Mermaid C4Context Diagram

```mermaid
C4Context
  title System Context: CPMS Platform

  Person(driver, "EV Driver", "Starts/stops sessions via mobile app or RFID")
  Person(operator, "CPO Operator", "Manages charge points via operator dashboard")
  [add other actors based on ui_scope decisions]

  System_Boundary(cpms_boundary, "CPMS Platform") {
    System(cpms, "CPMS Platform", "Cloud-native charge point management system.\nManages OCPP connections, sessions, billing, and roaming.")
  }

  System_Ext(charge_points, "OCPP Charge Points", "EV charging hardware.\nOCPP 1.6J / 2.0.1 over WebSocket (WSS)")
  System_Ext(ocpi_emsp, "OCPI eMSP Partners", "Roaming network partners.\nOCPI 2.1.1 / 2.2.1 REST")
  [add other external systems]

  Rel(driver, cpms, "Starts/stops sessions, views history", "HTTPS / GraphQL")
  Rel(operator, cpms, "Manages charge points, views analytics", "HTTPS / GraphQL")
  Rel(cpms, charge_points, "Sends commands, receives events", "OCPP over WSS")
  Rel(cpms, ocpi_emsp, "Pushes/pulls roaming data", "OCPI REST over HTTPS")
  [add other relationships]
```

Replace placeholder comments with actual actors/systems from requirements and decisions.
Ensure every actor and external system from the tables above appears in the diagram.

#### draw.io XML

Save the corresponding diagram as `output/c4-level-1-context/system-context.drawio`.
The draw.io XML must represent the same diagram as the Mermaid version, using draw.io's
built-in C4 shape library (namespace: `shape=mxgraph.c4.*`). Structure:
- Use `mxgraph.c4.person2` for human actors
- Use `mxgraph.c4.system` for the CPMS
- Use `mxgraph.c4.systemExternal` for external systems
- Use directional arrows with protocol labels for relationships

#### OCPP/OCPI Protocol Notes
Cite specific spec sections for every protocol boundary decision. Examples:
- "The CPMS acts as a CSMS (Central System Management System) per OCPP 2.0.1 §3.1.
  Charge Points initiate WebSocket connections to the CSMS endpoint."
- "OCPI 2.2.1 §3.1.1 defines the CPO and eMSP roles. This CPMS acts as a CPO platform,
  exposing OCPI endpoints to eMSP partners."
- "OCPP 1.6J §3.1: The Central System communicates with Charge Points using a
  WebSocket connection where the Charge Point is the WebSocket client."

#### Open Questions
List anything still unresolved that requires human input before Level 2.

#### Next Phase Preview
One paragraph: what Level 2 (Container) will add — which services, databases,
and queues will appear, and why Level 1 alone is not enough for implementation.

---

### ADR: `output/decisions/ADR-001-system-boundary.md`

Document the CPMS system boundary decision:
- What is inside the CPMS (owned and operated by this platform)
- What is outside (external systems the CPMS integrates with but does not own)
- Why OCPP CPs are external (the CPMS does not own the hardware)
- Why the payment gateway is external (PCI-DSS scope reduction)

---

## After Producing Output

Update `workflow-state.json`:
- `phases.c4_level_1_context.status = "completed"`
- `phases.c4_level_1_context.completed_at = <now>`
- `phases.c4_level_1_context.output_files = ["output/c4-level-1-context/system-context.md", "output/c4-level-1-context/system-context.drawio"]`
- `phases.c4_level_1_context.decisions_recorded = [list of key decisions]`
- Populate top-level `actors` array with all actors from the Actors Table
- Populate top-level `external_systems` array with all external systems
- Add `"ADR-001"` to `adr_index`
- `last_updated = <now>`

Save `workflow-state.json`.

Print:
```
Level 1 — System Context is complete.

Output files:
- output/c4-level-1-context/system-context.md
- output/c4-level-1-context/system-context.drawio
- output/decisions/ADR-001-system-boundary.md

Review the output, then run:
  /approve c4-context
to proceed to Level 2 (Container Diagram).
```
