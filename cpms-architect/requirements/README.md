# Requirements Folder

Place all project requirements documents here before running `/architect`.
The agent scans and reads every file in this folder automatically.

## Supported Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| Markdown | `.md` | Preferred format for new requirements |
| PDF | `.pdf` | Read automatically using the PDF tool |
| Word | `.docx` | Agent extracts text content |
| Plain text | `.txt` | Supported |

## What to Include

Add any of the following document types:

1. **Product Requirements Document (PRD)** — What the CPMS must do, feature list
2. **Stakeholder Requirements** — Who the system serves and their specific needs
3. **Non-Functional Requirements (NFR)** — Performance, availability, security targets
4. **Integration Requirements** — External systems the CPMS must connect to
5. **Regulatory / Compliance Requirements** — GDPR, PCI-DSS, local grid regulations
6. **Existing System Documentation** — If migrating from a legacy CPMS
7. **Business Constraints** — Budget envelope, team size, timeline, tech preferences
8. **OCPP / OCPI Protocol Requirements** — Which message types and modules are needed

## What NOT to Put Here

- Architecture documents → those go in `output/`
- OCPP/OCPI specification PDFs → those go in `docs/reference/`
- Code files

## Naming Convention

Use descriptive, lowercase, hyphenated filenames:

```
requirements/
  prd-cpms-v1.0.pdf
  nfr-performance-targets.md
  stakeholder-requirements.docx
  integration-requirements.md
  compliance-gdpr-pci.md
  ocpp-profile-requirements.md
```

## How to Use

1. Add one or more requirement files to this folder.
2. Open the project in Claude Code: `claude` (from the project root).
3. Run `/architect` to start the workflow.

The agent will:
- Read all files in this folder
- Summarise what each contributes
- Ask as many clarifying questions as needed (in batches) before producing any design output
