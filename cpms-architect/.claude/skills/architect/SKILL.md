---
name: architect
description: >
  Main entry point for all CPMS design work. Handles three design modes:
  (A) Full system design from scratch, (B) Scoped module or OCPP/OCPI profile design,
  (C) Incremental feature addition to an existing approved design.
  ALL modes use the same C4 model workflow (Context → Container → Component →
  Data Models → API Design) with human approval gates between each level.
  The agent asks as many clarifying questions as needed before starting any design.
tools:
  - Read
  - Write
  - Bash
---

# /architect Skill

## Purpose
Start or resume any CPMS design session. Always the first skill to run.

---

## Step 1 — Read State and Learnings

Read `workflow-state.json`. If it does not exist or `created_at` is empty,
initialise it (set `created_at` to current timestamp, all phases `locked` except
`intake` → `pending`).

Read `docs/learnings/architectural-patterns.md`. If non-empty, acknowledge:
"I have learnings from previous sessions — I'll apply them as we work."

---

## Step 2 — Determine Design Mode

Read `workflow-state.json` → `design_scope` and check existing phase statuses.

### If design_scope is already set and phases are in progress or approved:
Print a status summary:
```
Current design: [scope_name]
Mode: [Full System / Scoped Module / Incremental Feature]
Phases approved: [list]
Next phase: [pending phase] — run /[skill] to continue
```
Ask: "Do you want to continue the current design, start a new design, or add a
new feature on top of the existing approved design?"

### If no design is in progress:
Ask the human ONE initial question before loading anything else:

```
What would you like to design?

A) The full CPMS system from scratch
B) A specific OCPP/OCPI module or profile (e.g., Smart Charging, OCPI CDR)
C) A new feature to add on top of an existing approved design (e.g., Fleet Management)

Please describe what you have in mind, and I'll take it from there.
```

Wait for answer. Then proceed based on mode.

---

## Step 3A — Mode A: Full System Intake

1. Set `workflow-state.json` → `design_scope = "full_system"` and
   `scope_name = "CPMS Platform"`.

2. Scan and read all files in `requirements/`.
   - PDFs: use the PDF tool
   - .docx: use Read tool; if unreadable, ask human for text copy
   - .md / .txt: use Read tool
   If `requirements/` is empty: STOP and tell the human to add requirement files.

3. Scan `docs/reference/` and note available spec files. If empty, warn.

4. Load `agents/cpms-architect/INSTRUCTIONS.md`.

5. **First question batch** — present the standard baseline sections A through F
   from INSTRUCTIONS.md. Clearly label these as "Batch 1 of [N] — more may follow
   based on your answers."

6. Set `phases.intake.status = "in_progress"`, `started_at = <now>`,
   `clarifying_questions_asked = true`. Save. STOP and wait.

7. **After Batch 1 answers**: analyse the answers. For each answer:
   - Record decisions in `workflow-state.json` → `decisions`
   - Identify any ambiguities, contradictions, or dependencies that need more detail
   - Generate a **Batch 2** of follow-up questions for anything unclear
   - Repeat until you are confident you understand the full design space

8. **"Ready to Design" check**: when satisfied, present:
   ```
   ## Ready to Design — Confirmed Decisions

   | Decision | Value |
   |----------|-------|
   | OCPP versions | ... |
   | OCPI versions | ... |
   | [all major decisions] | ... |

   **Open questions** (non-blocking, will be flagged in each phase):
   - [list any]

   Are there any other constraints or requirements I should know before
   I begin Level 1 (System Context)?
   ```
   Wait for confirmation.

9. Set `phases.intake.status = "completed"`, `completed_at = <now>`,
   `clarifying_answers_received = true`. Save.
   Print: "Run `/approve intake` to confirm and unlock Level 1."

---

## Step 3B — Mode B: Scoped Module / Profile

1. Identify the module or profile from the human's description.
   Map it to a specification section (e.g., Smart Charging → OCPP 2.0.1 Part 2, §K).
   If unclear, ask: "Which module or profile exactly? And which spec version?"

2. Read `workflow-state.json`. If a full-system design exists and phases are approved,
   read the existing C4 documents as context for the scoped design.

3. Set `workflow-state.json` → `design_scope = "scoped_module"`,
   `scope_name = "<module name>"`.

4. Scan requirements for module-specific requirements (if any files in `requirements/`).

5. Ask the **module-specific questions** from INSTRUCTIONS.md for that module, PLUS
   a subset of the standard baseline questions relevant to this scope:
   - Section A questions 1, 3, 4, 5 (scale, versions, SLA, compliance)
   - Section B question 7 (relevant capabilities only)
   - All of Section C relevant to this module
   - Section F as applicable
   Skip sections not relevant to the module scope.
   Add any questions specific to this module beyond the standard list.

6. C4 scoping for modules:
   - **Level 1 (Context)**: the module's role within the broader CPMS and its
     external dependencies — not the full system context, but the module's boundary
   - **Level 2 (Container)**: the services and data stores involved in this module
   - **Level 3 (Component)**: internal structure of the module's primary service(s)
   - **Level 4 (Data Models)**: tables, Kafka schemas, Redis keys for this module only
   - **API Design**: gRPC methods and GraphQL types for this module

7. Output goes to `output/c4-level-*/` with a scope prefix in filenames
   (e.g., `output/c4-level-1-context/smart-charging-context.md`).

8. Follow the same approval gate sequence: `/approve intake` → `/c4-context` →
   `/approve c4-context` → etc.

---

## Step 3C — Mode C: Incremental Feature Addition

1. Ask: "What feature are you adding? Describe it in plain language including
   who benefits and what new capability it provides."

2. Read ALL existing approved C4 documents (listed in INSTRUCTIONS.md incremental
   section) as baseline context. Confirm: "I've read the existing architecture.
   Here's what I found: [brief summary of relevant existing design]."

3. Set `workflow-state.json` → `design_scope = "incremental"`,
   `scope_name = "<feature name>"`.
   Add entry to `workflow-state.json` → `features` map:
   ```json
   "<feature-slug>": {
     "status": "intake",
     "started_at": "<now>",
     "output_base": "output/features/<feature-slug>/"
   }
   ```

4. Ask the standard incremental feature questions from INSTRUCTIONS.md, plus
   any feature-specific questions needed. Ask in batches as needed.

5. Produce **Impact Assessment** (see INSTRUCTIONS.md) before any C4 work begins.
   Present it to the human and wait for explicit approval:
   "Do you approve this Impact Assessment? Are there impacts I've missed?"

6. After Impact Assessment approval: output goes to
   `output/features/<feature-slug>/c4-level-*/`.
   All delta documents reference and extend the existing baseline documents.
   Follow the same C4 phases and approval gates.

---

## Step 4 — Dynamic Follow-Up (all modes)

At any point during intake questioning, if the human's answer reveals:
- A new dependency not previously mentioned
- A conflict with the tech stack or an existing decision
- An ambiguity in scope (e.g., "maybe we need X" — ask for a firm answer)
- A requirement that implies a significantly more complex design

...add new questions to the next batch immediately. Do not ignore signals.
The intake phase is complete only when there are NO unresolved blocking questions.

---

## Important Rules

- Do NOT generate any diagram or architecture document during intake.
- Do NOT assume missing answers — ask.
- If a requirement contradicts the non-negotiable tech stack in CLAUDE.md, flag it
  explicitly and ask the human to confirm which takes precedence.
- The number of questions has no upper limit. Ask until you are confident.
