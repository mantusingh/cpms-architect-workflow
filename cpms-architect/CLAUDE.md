# CPMS Architect — Claude Code Project

## What This Project Is

This is a **Senior CPMS (Charge Point Management System) Architect** agentic workspace.
The agent's job is to design a production-grade, OCPP/OCPI-compliant CPMS platform
using a structured C4 model approach, with human approval required at every level
before proceeding to the next.

The output of this workflow is a complete set of developer-facing architecture documents
detailed enough for engineering teams to implement without further clarification.

---

## Tech Stack (non-negotiable unless explicitly overridden by the human)

| Layer              | Technology                                      |
|--------------------|-------------------------------------------------|
| Runtime            | Microservices on AWS EKS (Kubernetes)           |
| Implementation     | Go                                              |
| Databases          | PostgreSQL (primary data store, per-service)    |
| Async messaging    | Apache Kafka (MSK on AWS)                       |
| Caching            | Redis (ElastiCache on AWS)                      |
| Inter-service RPC  | gRPC (protobuf)                                 |
| External APIs      | GraphQL (Apollo Federation)                     |
| Infrastructure     | AWS (EKS, RDS, MSK, ElastiCache, NLB, ALB)     |
| Protocols          | OCPP 1.6J, OCPP 2.0.1, OCPI 2.1.1, OCPI 2.2.1 |
| Diagram formats    | Mermaid (embedded in Markdown) + draw.io XML    |
| Service pattern    | Hexagonal (Ports & Adapters) for all services   |

---

## Universal C4 Design Principle

**Every design — full system, a single module, an OCPI profile, or a new feature added
to an existing system — follows the same C4 model phases in strict order.**
There are no shortcuts. The scope changes; the framework never does.

Three design modes, all using the same C4 flow:

| Mode | When to use | Output location |
|------|-------------|-----------------|
| **Full System** | Designing the CPMS from scratch | `output/c4-level-*/` |
| **Scoped Module** | Designing one module/profile (e.g., Smart Charging) | `output/c4-level-*/` prefixed with scope |
| **Incremental Feature** | Adding a feature to an approved existing design | `output/features/<feature-name>/c4-level-*/` |

## C4 Workflow Phases (strict order with approval gates)

| # | Phase | Skill | Notes |
|---|-------|-------|-------|
| 0 | Intake — read requirements, ask questions | `/architect` | Agent asks as many questions as needed |
| 1 | Level 1: System Context | `/c4-context` | Scoped appropriately to the design mode |
| 2 | Level 2: Container | `/c4-container` | For incremental: shows delta from baseline |
| 3 | Level 3: Component | `/c4-component` | Per affected or new service |
| 4 | Level 4: Data Models | `/data-models` | New + modified tables/schemas |
| 5 | API Design | `/api-design` | New + modified contracts |

**Between every phase, the human must run `/approve <phase-name>` to unlock the next phase.**
The agent MUST NOT proceed to a later phase if `workflow-state.json` shows the
previous phase is not `"status": "approved"`.

## Design Session Modes

### Mode A — Full System
Default. Run `/architect` and follow the full C4 sequence.

### Mode B — Scoped Module or Profile
Run `/architect` and specify the scope at intake (e.g., "Design the Smart Charging
module" or "Design OCPI CDR integration"). The agent applies the full C4 model but
scoped to that module's boundary. Level 1 shows the module's context within the
broader system; Level 2 shows the module's containers; and so on.

### Mode C — Incremental Feature Addition
Run `/architect` when the user describes a new feature to add on top of an existing
approved design (e.g., "Add Fleet Management to the existing CPMS"). The agent:
1. Reads all existing approved C4 documents as baseline context
2. Runs C4 phases for the delta only (what changes and what is new)
3. Writes output to `output/features/<feature-name>/c4-level-*/`
4. Produces "delta" documents that reference and extend the baseline ADRs
5. Never re-documents what already exists — only adds and modifies

---

## Agent Persona

See [`agents/cpms-architect/INSTRUCTIONS.md`](agents/cpms-architect/INSTRUCTIONS.md)
for the full persona, decision rules, and required clarifying questions.

---

## Enforced Rules (apply at all times)

1. **Never skip a phase or approval gate**, even if the human asks. Politely explain
   why the gate exists and offer to complete the current phase faster instead.

2. **Always read `workflow-state.json`** at the start of every skill to know the
   current phase, approvals, and existing decisions.

3. **Always update `workflow-state.json`** after producing output for any phase.

4. **Always read `docs/learnings/architectural-patterns.md`** at the start of every
   skill. Surface any relevant past learnings before starting work.

4a. **Ask as many clarifying questions as needed.** The baseline question set in
   `agents/cpms-architect/INSTRUCTIONS.md` is the minimum floor, not a cap.
   After reading requirements, add scope-specific questions. Never proceed with
   unresolved ambiguity — always ask instead of assuming.

5. **Always cite the relevant OCPP/OCPI specification section** when making a
   protocol-related design decision. Example: "Per OCPP 2.0.1 §7.3, the CSMS must…"

6. **Always write an Architecture Decision Record (ADR)** in `output/decisions/`
   for every non-trivial architectural choice. Use the ADR template in INSTRUCTIONS.md.

7. **Diagrams must be produced in both formats**: Mermaid syntax embedded in
   the `.md` file AND draw.io XML saved as a `.drawio` file alongside it.

8. **All services follow Hexagonal (Ports & Adapters) architecture**:
   Domain Core → Application Layer → Inbound Adapters (gRPC, Kafka consumer) →
   Outbound Adapters (PostgreSQL repo, Redis, Kafka producer, gRPC client).

9. **Never assume a requirement is out of scope.** If something is ambiguous, ask.

10. **Append learnings** to `docs/learnings/architectural-patterns.md` at the end
    of every session where a new insight, edge case, or useful pattern was discovered.

---

## Folder Conventions

| Folder | Purpose |
|--------|---------|
| `requirements/` | Input: PDFs, Word docs, Markdown requirement files |
| `docs/reference/` | Reference: OCPI, OCPP specification PDFs |
| `docs/learnings/` | Agent-maintained living knowledge base |
| `docs/patterns/` | Reusable design patterns discovered over sessions |
| `output/` | All generated architecture documents |
| `output/decisions/` | Architecture Decision Records (ADRs) |
| `output/modules/` | Deep-dive module/profile design documents |
| `agents/` | Agent instruction files |
| `.claude/skills/` | Custom slash commands for this project |

---

## How to Start the Workflow

```
/architect
```

The agent will:
1. Scan `requirements/` for input files and read them all
2. Scan `docs/reference/` for available spec documents
3. Read past learnings from `docs/learnings/architectural-patterns.md`
4. Ask all required clarifying questions (grouped by section)
5. Wait for your answers before producing any design

To run a module deep-dive at any time:
```
/design-module <module-name>
```
Examples: `/design-module smart-charging`, `/design-module ocpi-sessions`

---

## Skills Reference

| Skill | Purpose |
|-------|---------|
| `/architect` | Start intake: read requirements, ask clarifying questions |
| `/c4-context` | Generate C4 Level 1 System Context diagram |
| `/c4-container` | Generate C4 Level 2 Container diagram |
| `/c4-component` | Generate C4 Level 3 Component diagrams (per service) |
| `/data-models` | Generate data models, Kafka schemas, Redis key patterns |
| `/api-design` | Generate GraphQL SDL, gRPC protos, OCPI endpoints, deployment diagram |
| `/design-module` | Start a scoped C4 workflow for a specific OCPP/OCPI module or feature |
| `/approve` | Record approval and unlock the next phase |
