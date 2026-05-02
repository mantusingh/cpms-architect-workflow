# Incremental Feature Designs

This folder contains C4 delta documents for features added on top of an existing
approved CPMS design. Each feature gets its own subfolder following the same
C4 structure as the baseline design.

## Folder Structure

```
output/features/
  <feature-slug>/
    c4-level-1-context/      # Scoped Context diagram (delta)
    c4-level-2-container/    # Scoped Container diagram (new + modified)
    c4-level-3-component/    # Component diagrams for new/modified services
    c4-level-4-data-models/  # New tables, schemas, Redis keys
    api-design/              # New gRPC methods, GraphQL types, OCPI endpoints
    impact-assessment.md     # Impact on existing services — approved before C4 starts
```

## How to Start an Incremental Design

Run `/architect` or `/design-module <feature-name>` when you want to add a feature
to an existing approved design. The agent will:

1. Detect that approved phases exist
2. Read all existing C4 documents as baseline context
3. Ask incremental-specific clarifying questions (plus as many follow-ups as needed)
4. Produce an Impact Assessment for human approval before any C4 work begins
5. Run the same C4 phases scoped to the feature delta

## Delta Document Conventions

Every delta document:
- Opens with a **Baseline Reference** line citing the existing document it extends
- Marks new elements as **NEW**
- Marks changed elements as **MODIFIED** with explanation of what changes and why
- Does NOT re-document unchanged parts — references the baseline by file path
- Produces new ADRs for decisions that affect the existing design (stored in
  `output/decisions/` alongside baseline ADRs, numbered sequentially)
