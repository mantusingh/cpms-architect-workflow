# Architecture Decision Records (ADRs)

This folder contains Architecture Decision Records for all non-trivial design choices
made during the CPMS architecture workflow.

## ADR Naming Convention

`ADR-NNN-<short-slug>.md`

Example: `ADR-003-kafka-event-backbone.md`

## ADR Template

Each ADR follows this structure:

```markdown
# ADR-NNN: [Short Title]
**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded
**Phase**: [C4 level or module that produced this ADR]

## Context
[Why this decision is needed — the problem or constraint]

## Decision
[What was decided — clearly stated]

## Rationale
[Why this was chosen — trade-offs, alignment with requirements]

## Alternatives Considered
| Alternative | Reason Rejected |
|-------------|-----------------|
| ...         | ...             |

## Consequences
[Positive and negative consequences of this decision]

## OCPP/OCPI References
[Relevant specification sections, if applicable]
```

## Expected ADR Index

| ADR | Title | Phase |
|-----|-------|-------|
| ADR-001 | System Boundary | Level 1 |
| ADR-002 | OCPP Gateway Separation | Level 2 |
| ADR-003 | Kafka Event Backbone | Level 2 |
| ADR-004 | Redis Connection Registry | Level 2 |
| ADR-005 | GraphQL Federation | Level 2 |
| ADR-006 | Service Decomposition | Level 2 |
| _(more added per phase)_ | | |

## Superseded ADRs

When a decision is revised, the original ADR is updated to `Status: Superseded`
with a reference to the new ADR number. Old ADRs are never deleted.
