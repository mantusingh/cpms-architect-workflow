---
name: approve
description: >
  Record human approval for a completed workflow phase and unlock the next phase.
  Usage: /approve <phase-name>
  Valid phase names: intake, c4-context, c4-container, c4-component, data-models, api-design
  Optionally append notes: /approve c4-context -- looks good but simplify the OCPI boundary
tools:
  - Read
  - Write
---

# /approve Skill

## Usage

```
/approve <phase-name> [-- optional notes]
/approve impact-assessment [-- optional notes]   (incremental mode only)
```

**Valid phase names:**
- `intake`
- `impact-assessment` _(incremental feature mode only — approves the Impact Assessment)_
- `c4-context`
- `c4-container`
- `c4-component`
- `data-models`
- `api-design`

---

## Phase Mapping

| Argument | State key | Next phase key | Next skill |
|----------|-----------|----------------|------------|
| `intake` | `intake` | `c4_level_1_context` | `/c4-context` |
| `impact-assessment` | _(feature-scoped)_ | `c4_level_1_context` | `/c4-context` |
| `c4-context` | `c4_level_1_context` | `c4_level_2_container` | `/c4-container` |
| `c4-container` | `c4_level_2_container` | `c4_level_3_component` | `/c4-component` |
| `c4-component` | `c4_level_3_component` | `c4_level_4_data_models` | `/data-models` |
| `data-models` | `c4_level_4_data_models` | `api_design` | `/api-design` |
| `api-design` | `api_design` | _(final phase)_ | _(workflow complete)_ |

For **incremental feature mode** (`design_scope == "incremental"`), all phase state
is stored under `workflow-state.json` → `features.<feature-slug>.phases` rather than
the top-level `phases` object. The `/approve` skill checks `design_scope` and reads
from the correct location.

---

## Steps

1. Read `workflow-state.json`.

2. Map the phase argument to the correct state key from the table above.
   If the argument is not in the valid list, print the valid options and stop.

3. Verify `phases.<phase_key>.status == "completed"`.
   If the status is not `"completed"`:
   - If `"locked"`: tell the human the previous phase must be completed and approved first.
   - If `"pending"` or `"in_progress"`: tell the human which skill to run to complete
     this phase (e.g., "Run `/c4-context` to generate the Level 1 output first.").
   - If `"approved"`: tell the human this phase is already approved.

4. Parse optional notes from after the `--` separator (if present).

5. If notes contain a **revision request** (keywords: "change", "update", "revise",
   "remove", "add", "fix", "simplify"), offer two options:
   ```
   You've requested a revision: "[note text]"

   Options:
   A) Approve now and record the revision as a task for the developer team
      in output/decisions/revision-requests.md — the workflow advances.
   B) Hold approval — I'll address the revision in this phase before approving.
      Re-run the relevant skill, then /approve again.

   Which do you prefer? (A or B)
   ```
   Wait for the human to choose before proceeding.

6. For a straightforward approval (no revision):
   - Set `phases.<phase_key>.status = "approved"`
   - Set `phases.<phase_key>.approved_at = <current timestamp>`
   - If notes provided, set `phases.<phase_key>.notes = <notes>`
   - Set `phases.<next_phase_key>.status = "pending"` (unlocking it)
   - Set `current_phase = <next_phase_key>`
   - Set `last_updated = <now>`
   - Save `workflow-state.json`.

7. Print a clear confirmation:

```
APPROVED: [Phase Display Name]
Approved at: [timestamp]
Notes recorded: [notes if any, else "none"]

UNLOCKED: [Next Phase Display Name]
Run /[next-skill] to begin.
```

   If this was the final phase (`api-design`), print instead:
```
APPROVED: API Design
Approved at: [timestamp]

WORKFLOW COMPLETE
All 6 phases have been approved. Here is the full output index:

[List all files in output/ with one-line descriptions]

The architecture is ready for implementation.
```
