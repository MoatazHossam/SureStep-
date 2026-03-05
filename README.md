# SureStep-
This App will produce the Fit Gap and FDD automatically
# Dynamics 365 FRD → Fit-Gap + FDD + TDD + Backlog (AI-driven System)
Owner goal: Upload an FRD (Word/PDF) and automatically generate:
1) Fit-Gap Excel workbook
2) FDD documents (one per GAP)
3) TDD drafts (one per GAP)
4) Development backlog in GitHub (Issues + optional GitHub Project)
5) Traceability matrix linking ReqId → FitGap row → FDD/TDD → Issue

## 0) Target users
Dynamics 365 Finance & Operations consultants using Sure Step / similar documentation:
- Functional Consultant / Functional Lead
- Solution Architect
- Technical Architect / Developers
- PM / PMO

## 1) Non-negotiable rules (quality + safety)
- Do NOT invent (hallucinate) Dynamics features or project facts.
- If information is missing/unclear, mark `NeedsClarification=true` and create an Open Question.
- Every requirement MUST have a stable ReqId.
- Every GAP MUST produce exactly: 1 FDD + 1 TDD + 1 backlog Issue.
- Every artifact must include ReqId and FitGapId references.
- Outputs must be deterministic enough to pass unit tests.

## 2) Repo deliverables (what must exist in main branch)
### A) Folder structure
/
  SPEC.md
  README.md
  pyproject.toml  (or requirements.txt)
  .env.example
  /src
    main.py
    pipeline.py
    frd_ingest.py
    requirement_mining.py
    capability_library.py
    fitgap_engine.py
    excel_writer.py
    doc_writer.py
    backlog_github.py
    traceability.py
    schemas.py
    utils.py
  /templates
    fit_gap_template.xlsx
    FDD_template.docx
    TDD_template.docx
  /knowledge
    d365_capabilities.md
    d365_patterns.md
    surestep_artifacts.md
  /output  (gitignored)
  /tests
    test_schema_validation.py
    test_excel_structure.py
    test_doc_structure.py
    test_traceability.py
  /sample
    sample_frd.docx (optional) or sample_frd.pdf (optional)

### B) Tech stack (must use Python)
- Python 3.11+
- Document parsing:
  - docx: python-docx
  - pdf: pdfplumber (fallback), plus optional integration placeholder for Azure Document Intelligence
- Excel generation: openpyxl
- HTTP: requests
- CLI: typer (or argparse)
- Testing: pytest
- Config: pydantic-settings (or dotenv)

### C) Execution modes
1) Local CLI:
   - `python -m src.main run --frd ./path/to/FRD.docx --out ./output`
2) GitHub workflow (optional later):
   - Trigger on FRD added to repo or uploaded to a folder (not required in v1)

## 3) Inputs
### A) FRD
- Word (.docx) or PDF (.pdf)
- May contain headings, tables, bullet points, and mixed narrative

### B) Templates (must be used)
- `/templates/fit_gap_template.xlsx` defines required columns and sheets
- `/templates/FDD_template.docx` defines required sections and headings
- `/templates/TDD_template.docx` defines required sections and headings

### C) Knowledge base (local files in /knowledge)
Start simple in v1 (no external scraping):
- `d365_capabilities.md`: a curated feature catalog by module
- `d365_patterns.md`: standard patterns for workflows, security, data entities, integrations
- `surestep_artifacts.md`: required artifact sections and document control rules

## 4) Core data contracts (JSON schemas)
All intermediate data must be produced as JSON in `/output/_intermediate/` and validated.

### A) requirements.json (array)
Each entry:
- reqId: "REQ-0001"
- title: short
- text: full requirement statement
- moduleGuess: e.g., "Procurement and sourcing"
- processGuess: e.g., "Procure-to-Pay"
- priority: "Must/Should/Could/Won't" (or Unknown)
- sourceRef: { file, sectionHeading, pageOrParagraphIndex }
- needsClarification: bool
- clarificationQuestions: string[]

### B) fitgap.json (array)
- fitGapId: "FG-0001"
- reqId
- status: "Fit" | "Gap" | "Partial" | "NeedsClarification"
- mappedCapabilities: [{ capabilityId, name, rationale, confidence(0..1) }]
- solutionApproach: "Config" | "Customization" | "Integration" | "ISV" | "Reporting" | "DataMigration" | "Other"
- summaryConfig: string
- summaryGap: string
- assumptions: string[]
- openQuestions: string[]
- complexity: "S" | "M" | "L" | "XL"
- estimateHours: number (can be 0 if unknown)
- ownerRole: "Functional" | "Technical" | "Integration" | "Data" | "Reporting"
- statusWorkflow: "Draft" | "Reviewed" | "Approved"

### C) artifacts.json
- reqId
- fitGapId
- fddId: "FDD-0001"
- tddId: "TDD-0001"
- fddPath
- tddPath

### D) backlog.json
- reqId
- fitGapId
- issueNumber (after creation)
- issueUrl
- labels: string[]
- milestone: string | null
- projectItemId (optional)

## 5) Outputs (must be generated)
### A) Fit-Gap Excel
Path: `/output/FitGap.xlsx`
Must contain sheets:
1) "FitGap"
2) "OpenQuestions"
3) "Traceability"

FitGap columns (exact header names required):
- ReqId
- Process
- Module
- Requirement
- FitGapStatus
- StandardFeatureMapping
- SolutionApproach
- ConfigurationSummary
- GapSummary
- IntegrationNeeded
- ReportingNeeded
- DataMigrationImpact
- Assumptions
- OpenQuestions
- Complexity
- EstimateHours
- OwnerRole
- WorkflowStatus
- FDDId
- TDDId
- GitHubIssue

OpenQuestions columns:
- ReqId
- Question
- OwnerRole
- Status

Traceability columns:
- ReqId
- FitGapId
- FDDId
- TDDId
- GitHubIssue

### B) FDD documents (one per GAP/PARTIAL requiring work)
Folder: `/output/FDD/`
File naming: `FDD-0001_REQ-0001.docx`
Required headings (must exist as paragraphs with these titles):
1. Document Control
2. Business Scenario
3. In Scope
4. Out of Scope
5. Assumptions and Dependencies
6. To-Be Process
7. Functional Requirements
8. Solution Design
9. Security Design
10. Reporting Design
11. Integration Design
12. Data Migration
13. Acceptance Criteria and Test Scenarios
14. Open Issues

### C) TDD drafts (one per GAP/PARTIAL requiring work)
Folder: `/output/TDD/`
File naming: `TDD-0001_REQ-0001.docx`
Required headings:
1. Overview
2. Architecture and Approach
3. D365FO Objects and Extensions
4. Data Model Details
5. Business Logic (Pseudo-code)
6. Security Artifacts
7. Integrations (Implementation)
8. Performance and Reliability
9. Logging and Error Handling
10. Testing Approach
11. Deployment Notes
12. Task Breakdown and Estimates

### D) GitHub backlog
Create GitHub Issues using GitHub REST API.
- One Issue per GAP/PARTIAL (not for FIT unless configured)
Issue title format:
`[REQ-0001] <short title>`

Issue body must include:
- Requirement text
- Fit/Gap status and approach
- Links or paths to FDD/TDD outputs
- Acceptance criteria bullets (from FDD section)

Labels:
- Module:<moduleGuess>
- Type:Gap (or Type:Partial)
- Approach:<solutionApproach>

Optional: add to GitHub Project if env var provided.

## 6) AI behavior (LLM boundaries)
Implement a clean abstraction for LLM calls:
- `LLMProvider` interface with method `generate_json(prompt, schema)` and `generate_text(prompt)`
- Must support:
  - Local "mock mode" for tests (deterministic)
  - Real mode with OpenAI API key (env var `OPENAI_API_KEY`)

In v1, tests must run in mock mode and still generate valid outputs.

## 7) Configuration (env vars)
Create `.env.example` with:
- OPENAI_API_KEY=...
- OPENAI_MODEL=gpt-5.2-codex (default if available)
- GITHUB_TOKEN=...
- GITHUB_OWNER=...
- GITHUB_REPO=...
- GITHUB_PROJECT_NUMBER=... (optional)
- OUTPUT_DIR=./output
- GENERATE_FIT_ISSUES=false (default)
- RUN_MODE=mock|real  (default mock)

Note: Model name may vary; code must allow override.

## 8) CLI commands
### A) Run full pipeline
`python -m src.main run --frd <path> --out <output_dir>`

### B) Only parse FRD and extract requirements
`python -m src.main extract --frd <path> --out <output_dir>`

### C) Only generate Fit-Gap from extracted requirements
`python -m src.main fitgap --out <output_dir>`

### D) Generate docs from fitgap.json
`python -m src.main docs --out <output_dir>`

### E) Publish backlog to GitHub
`python -m src.main backlog --out <output_dir>`

## 9) Acceptance tests (must pass)
- Validate JSON files match schema contracts.
- Verify Excel:
  - file exists
  - required sheets exist
  - first row headers match exact column names
  - at least 1 row is written for a sample requirement set
- Verify Word docs:
  - files exist for each GAP
  - required headings exist
- Verify traceability:
  - every GAP row has FDDId, TDDId
  - backlog.json includes issue numbers if backlog step executed in integration tests (can be skipped in CI)
- Provide a `sample_requirements.json` fixture for tests (so no FRD is required for CI).

## 10) Implementation notes (important)
- Keep generation reliable by using structured prompts and JSON schema validation.
- Provide a simple rule-based fallback if LLM returns invalid JSON (retry with “fix JSON to match schema” prompt).
- Always include ReqId, FitGapId, FDDId, TDDId in every artifact.
- Add logging at each step; write progress to `/output/run_log.txt`.

---

# RUN PLAN FOR CODEX (execute as separate PRs)
Codex: implement the following PRs in order. Each PR must include tests and docs updates.

## PR1 — Repo scaffold + templates + schemas + tests skeleton
- Create folder structure, CLI skeleton, config loading, logging
- Add templates:
  - Create a minimal `fit_gap_template.xlsx` with the required sheets/headers
  - Create minimal `FDD_template.docx` and `TDD_template.docx` with required headings
- Add JSON schema models in `src/schemas.py`
- Add pytest skeleton and fixtures
- Ensure `pytest` passes (mock mode)

## PR2 — FRD ingest (docx + pdf) + requirement extraction (mock LLM)
- Implement `frd_ingest.py`:
  - docx: extract headings + paragraphs
  - pdf: extract text by page (best effort)
- Implement `requirement_mining.py`:
  - In mock mode, parse with heuristics: treat bullet lines as requirements
  - Output `requirements.json` validated

## PR3 — Knowledge loader + capability mapping
- Implement `knowledge/` file loaders
- Implement naive capability search (keyword match) for v1
- Produce `mappedCapabilities` with confidence scores

## PR4 — Fit-Gap engine + Fit-Gap Excel generator + traceability sheet
- Implement `fitgap_engine.py` rules:
  - If capability confidence >= 0.75 and no customization keywords → Fit
  - else Gap/Partial; generate open questions when needed
- Write `fitgap.json`
- Generate `/output/FitGap.xlsx` using the template
- Add tests for Excel structure

## PR5 — FDD/TDD document generators
- Generate one FDD/TDD per GAP/PARTIAL
- Ensure headings exist
- Update FitGap.xlsx with FDDId/TDDId columns
- Add tests for required headings

## PR6 — GitHub backlog publisher
- Implement creating issues via REST API (token + owner/repo)
- Add labels, link artifacts
- Update traceability sheet and write `backlog.json`
- Make network calls optional; tests should mock GitHub API

## PR7 — Real LLM integration (optional, behind RUN_MODE=real)
- Add OpenAI API integration only in real mode
- Keep tests in mock mode
- Ensure robust JSON validation + retry

Done.

## PR automation troubleshooting (`Failed to create PR`)
If Codex shows **"Failed to create PR"**, use this preflight checklist before calling `make_pr`:

1. Confirm branch context
   - `git branch --show-current` is not detached
   - you are on the working branch intended for the PR
2. Confirm repository state
   - `git status --short` is clean **after** staging/commit
   - `git log --oneline -1` shows the new commit you expect
3. Confirm command order
   - Required sequence: **edit -> test/check -> git commit -> make_pr**
   - Do **not** call `make_pr` before commit
   - Do **not** finish with committed changes without calling `make_pr`

Common invalid states that cause PR automation failure:
- State A: `make_pr` called with no new committed changes
- State B: changes committed, but `make_pr` never called

Minimal recovery steps:
1. `git status --short && git branch --show-current`
2. If needed: `git add -A && git commit -m "<message>"`
3. Call `make_pr` once with a clear title/body that summarizes the committed diff
