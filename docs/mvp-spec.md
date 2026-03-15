# MVP Specification (Re-Baselined): UI-Configured Real Document PDF Generator

## 1. Objective
Deliver a Windows desktop executable that lets QA users configure document package rules in UI and generate real PDF files (invoice, shipping docs, bank statement, and related evidence) with deterministic linkage and reproducibility.

This is the main product target.

## 2. Why This Re-Baseline
The current implementation is a useful engine foundation, but it is not yet the final QA-facing outcome.

Current state:
- Generates deterministic synthetic structured data and manifests.
- Produces JSON artifacts for linkage validation.
- Provides desktop execution flow.

Target state:
- Generate real, human-readable PDF documents according to user-defined page composition and linkage constraints.

## 3. End-User Product Requirement (Authoritative)
The user of this app must be able to configure in UI:
- Total document count.
- Per-document page count.
- Per-page or page-range content type (invoice, shipping, bank, etc).
- Interleaving rules (for example, page 1 and 3 of invoice A, page 2 of invoice B).
- Linkage constraints across pages and docs (for example, PO 111 must appear in invoice and shipping docs).

When user clicks Generate Documents, the app must output real PDFs that follow the configured plan.

## 4. Scope for Final MVP
### 4.1 In-Scope Output
- Real PDF files for at least:
  - Invoice
  - Shipping document family (PO, BOL, shipping)
  - Bank statement pages
  - Receipt and credit memo as needed by scenario
- One package manifest containing:
  - Canonical field values
  - Linkage graph
  - Page-to-entity mapping
  - Excel-comparison expected values

### 4.2 In-Scope UI
- Document plan builder (document count, page plan, linkage rules).
- Basic template/style choices.
- Deterministic repeatability number (seed).
- Generate button for full package creation.

### 4.3 Out of Scope for This MVP
- OCR noise simulation.
- Pixel-perfect replicas of every external source format.
- LLM-driven narrative content generation.

## 5. Functional Requirements
1. Deterministic generation:
- Same plan plus same seed must reproduce same outputs.

2. Page-level composition:
- User can map pages to logical entities (invoice A/B, shipping doc, bank pages, etc).

3. Interleaving support:
- Non-contiguous pages for same entity are supported.

4. Cross-document linkage:
- IDs and field values remain consistent where configured.
- Controlled mismatches can be injected where configured.

5. Real PDF rendering:
- Output is PDF files (not JSON-only artifacts).

6. Validation assets:
- Ground-truth manifest remains available for automated QA checks.

## 6. Re-Baselined Delivery Plan and Status
### Phase 0: Foundation Engine (Completed)
Status: Completed

Completed items:
- Deterministic seed management.
- Request/schema validation.
- Scenario builders (happy path and negative variants).
- Linkage validator and quality checks.
- Variation planner and diversity checks.
- Package exporter and run reports.
- Desktop executable scaffolding and build pipeline.

Note:
- This phase outputs structured data and manifests, not final PDF business documents.

### Phase 1A: JSON-Based Document Plan Foundation (Completed)
Status: Completed

Completed in Increment 1 (User-Verified):
- Added `GenerationPlan` schema for page-level composition with coverage and overlap validation.
- Added CLI plan validation modes: `--plan-json`, `--plan-example`, `--plan-stdin`.
- Added interleaved plan example at `examples/document-plan-interleaved.json`.
- Added stdin safety guard so `--plan-stdin` and `--request-stdin` fail fast with guidance when no input is piped.

Completed in Increment 2 (User-Verified):
- Added UI Document Plan panel for page-composition JSON editing and validation.
- Added `Validate Plan` and `Load Example Plan` actions in desktop UI.
- Added clear plan-valid and plan-invalid user feedback flow in UI status/output.
- Added simplified launch convenience script (`run_app.ps1`) and verified launch path options.

### Phase 1B: Form-Based Plan Builder UX (Deferred)
Status: Deferred

Planned deliverables:
- Non-technical form controls for:
  - total documents,
  - per-document page definitions,
  - interleaving and linkage constraints.
- Reduced need for raw JSON editing in day-to-day QA usage.

Sequencing note:
- Phase 1B is intentionally deferred until after a usable Phase 2 PDF rendering slice exists.
- This allows end-to-end value delivery earlier (real PDF generation), then UX hardening on top.

### Phase 2: PDF Rendering Layer (In Progress)
Status: In Progress

Completed in Increment 1 (User-Verified):
- Added a working plan-to-PDF rendering vertical slice that generates real PDF files.
- Added CLI render mode: `--render-plan-pdf` with `--plan-json`, `--plan-example`, and `--plan-stdin` input options.
- Added UI action: `Generate PDFs From Plan` using the validated plan editor payload.
- Verified generated PDFs open correctly and include expected page-level block content.

Completed in Increment 2 (User-Verified):
- Added optional merged package PDF output in plan order (`package-merged.pdf`).
- Added CLI merge option: `--merge-plan-pdf` for combined package output.
- Added UI control to enable merged package output from the plan generation flow.
- Verified merged package output path and generated merged PDF via UI run.

Completed in Increment 3 (User-Verified):
- Expanded content-specific template sections for `receipt` and `credit_memo` pages.
- Added explicit receipt and credit memo line rendering in generated PDF output.
- Verified receipt and credit memo template lines appear correctly in generated PDFs.

Deliverables:
- Renderers for invoice, shipping docs, bank statements, receipt, credit memo.
- Multi-page and continuation rules.
- Template packs for business-like visual outputs.

### Phase 3: Composition Orchestrator (In Progress)
Status: In Progress

Completed in Increment 1 (User-Verified):
- Added first composition orchestration slice that combines plan-ordered pages into one merged package PDF.
- Kept per-document PDF outputs alongside merged package output for QA workflows.

Completed in Increment 2 (User-Verified):
- Added `plan-linkage-summary.json` output for page-level mapping and linkage metadata.
- Added entity-level link consistency checks across rendered pages and merged output context.
- Exposed linkage summary path and consistency result in CLI and UI completion feedback.
- Verified linkage summary generation and consistency status in runtime output.

Completed in Increment 3 (User-Verified):
- Added configurable linkage rule profiles in plan rendering: `strict`, `relaxed`, and `mismatch_injection`.
- Added CLI support for linkage profile selection via `--linkage-mode`.
- Added UI linkage mode control and included selected mode in completion feedback.
- Verified linkage profile behavior and runtime output for selected modes.

Completed in Increment 4 (User-Verified):
- Added advanced merged page composition modes: `document_order`, `content_type_round_robin`, and `entity_round_robin`.
- Added CLI composition selection via `--composition-mode`.
- Added UI composition mode control and completion feedback for selected strategy.
- Verified runtime output and merged generation behavior for selected composition mode.

Completed in Increment 5 (User-Verified):
- Added UI consistency-failure guidance with actionable hints per offending linkage key/entity.
- Added mismatch-injection guidance to direct users to strict or relaxed linkage mode.
- Surfaced guidance directly in completion feedback when consistency issues are present.
- Verified guidance rendering and runtime behavior in mismatch injection mode.

Completed in Increment 6 (User-Verified):
- Added actionable grouped linkage issue summaries in `plan-linkage-summary.json`.
- Added grouped sections for inconsistent values and injected mismatches by link key.
- Added recommended actions per grouped issue to speed QA triage.
- Verified grouped summary payload content in generated output manifest.

Deliverables:
- Map plan blocks to rendered pages.
- Merge ordered pages into final PDFs per configured document.
- Guarantee page-level linkage consistency.

### Phase 4: Final QA-Ready EXE Workflow (Pending)
Status: Pending

Deliverables:
- End-to-end Generate Documents flow in UI.
- Real PDF outputs + manifest bundle.
- Friendly user messaging for failures and corrective action.

### Phase 5: CI/CD and Release Hardening (Partially Completed)
Status: In Progress

Completed:
- Build script and packaging baseline.
- JFrog-oriented dependency path integration.

Pending:
- Final CI workflow for full product behavior.
- Release runbook for QA users.

## 7. Acceptance Criteria for Final MVP
1. User can configure a multi-document, multi-page plan in UI.
2. User can define interleaving and linkage constraints in that plan.
3. Clicking Generate Documents creates real PDFs matching the plan.
4. Linkage and configured mismatches appear exactly as configured.
5. Ground-truth manifest is generated for automation.
6. Same seed and plan reproduces the same package.
7. Packaged executable runs this flow without technical setup.

## 8. Immediate Next Implementation Steps
1. Keep per-document PDF outputs and manifests synchronized for QA validation.
2. Add export-friendly summary views for inconsistency triage in QA runs.
3. Add summary-level severity tags to improve QA filtering/reporting.
4. Add UI quick filters for mismatch-only versus true consistency conflicts.
5. Return to Phase 1B form-based plan builder UX after Phase 2 and Phase 3 slices are user-verified.

## 9. Project Direction Lock
This document supersedes earlier interpretations that focused on JSON-only generation as the end product.

JSON generation remains a core internal validation layer, but the deliverable product for QA is the PDF-generating executable described above.
