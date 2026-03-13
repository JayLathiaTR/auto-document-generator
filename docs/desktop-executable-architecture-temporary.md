# Temporary Architecture Note: Desktop Executable Flow

## Status
This is an internal planning document for team understanding.

Remove this document after implementation is complete and the final product documentation is published.

## Goal
Deliver a Windows desktop executable that generates deterministic synthetic audit document packages locally, without requiring GitHub Pages or a backend API.

## Why Desktop for This Project
- Keeps Python deterministic generator as-is.
- Supports offline usage for QA teams.
- Avoids API hosting and runtime key management.
- Simplifies enterprise deployment in controlled environments.

## MVP Desktop Stack
1. UI framework: PySide6
2. Generator engine: existing deterministic Python modules
3. Packaging: PyInstaller
4. Output artifacts: local folder containing rendered documents and manifests

## High-Level Runtime Architecture
1. User configures generation in desktop form.
2. UI submits request to application service layer.
3. Service layer validates request and builds run plan.
4. Deterministic engine generates canonical truth, linkage graph, and variation plan.
5. Renderers produce artifact files.
6. Manifest writer emits ground-truth files.
7. UI displays run summary and opens output folder.

## Proposed Project Structure

```text
auto-document-generator/
  app/
    ui/
      main_window.py
      dialogs/
      widgets/
    services/
      request_service.py
      run_service.py
      export_service.py
    state/
      app_state.py
  engine/
    models/
      documents.py
      scenario.py
      linkage.py
    deterministic/
      seed_manager.py
      id_generator.py
      date_generator.py
      amount_generator.py
    variation/
      planner.py
      profiles.py
      diversity_validator.py
    scenarios/
      happy_path.py
      partial_match.py
      key_mismatch.py
      amount_mismatch.py
      date_anomaly.py
      multi_page_invoice.py
    renderers/
      invoice_renderer.py
      po_renderer.py
      bol_renderer.py
      shipping_renderer.py
      receipt_renderer.py
      credit_memo_renderer.py
      bank_entry_renderer.py
    manifests/
      ground_truth_writer.py
      linkage_graph_writer.py
      metadata_writer.py
      variation_plan_writer.py
  config/
    variation/
      layout_families.json
      label_sets.json
      date_formats.json
      weights.json
    defaults.json
  outputs/
    .gitkeep
  scripts/
    build_exe.ps1
  requirements.txt
  pyproject.toml
  README.md
```

## UI Screens (MVP)
1. Run Configuration
- Seed
- Scenario type
- Document counts
- Multi-page options
- Diversity policy knobs
- Output directory

2. Run Progress
- Step-by-step status (validation, generation, rendering, manifest)
- Cancel support (safe stop between phases)

3. Results
- Generated package path
- Summary card: docs created, scenario type, linkage status
- Buttons: open folder, run again with same seed, export run config

## Core Request Contract (Desktop -> Engine)

```json
{
  "seed": 20260313,
  "scenario_type": "happy_path",
  "document_count": {
    "invoice": 1,
    "po": 1,
    "bol": 1,
    "shipping_document": 1,
    "receipt": 1,
    "credit_memo": 0,
    "bank_entry": 1
  },
  "multi_page": {
    "enabled": true,
    "target_document": "invoice",
    "page_count": 2,
    "omit_invoice_id_on_continuation": true
  },
  "diversity_policy": {
    "enforce_min_layout_families": 2,
    "enforce_min_label_sets": 2,
    "max_single_layout_ratio": 0.6
  },
  "output_directory": "C:/Users/<user>/Documents/AuditSamples"
}
```

## Packaging and Build Flow

## Dependencies
Keep dependencies minimal and pinned.

Example `requirements.txt` (initial):
- PySide6
- pydantic
- faker
- reportlab
- jinja2

## PyInstaller Command (Baseline)

```powershell
pyinstaller --noconfirm --clean --name audit-doc-generator --windowed app/ui/main_window.py
```

Add data files in spec or command options:
- `config/variation/*.json`
- UI assets
- template files used by renderers

## Recommended Build Script
Create `scripts/build_exe.ps1` to:
1. Clean previous build artifacts.
2. Install pinned dependencies.
3. Run tests.
4. Build executable with PyInstaller.
5. Copy config/templates to distribution folder.
6. Write build metadata file (version, commit SHA, build time).

## Distribution Plan (Internal)
1. Produce zip package from `dist/audit-doc-generator/`.
2. Publish to internal file share or release page.
3. Include release notes and known limitations.

## Operational Guardrails
1. All generation remains local to user machine.
2. No external API calls in MVP.
3. No runtime secrets required.
4. Output folder must be user-configurable.
5. Application logs must avoid sensitive real-world data.

## Testing Strategy (Desktop)
1. Unit tests
- Seed determinism
- Linkage integrity
- Scenario-specific mismatch behavior
- Variation diversity checks

2. Integration tests
- End-to-end generation from request to output package
- Manifest correctness against generated artifacts

3. Desktop smoke tests
- App launch
- Run success for each scenario type
- Output folder open action works

## Risks and Mitigations
1. Risk: Large executable size
- Mitigation: trim dependencies and assets

2. Risk: Renderer complexity for multi-page documents
- Mitigation: start with one invoice renderer style, then expand via variation layer

3. Risk: Non-deterministic behavior introduced by UI threading
- Mitigation: isolate RNG in engine; UI only orchestrates

## Phased Implementation Plan
1. Phase 1: Engine foundation
- Deterministic generators, linkage, manifests

2. Phase 2: Minimal PySide6 UI
- Single run form + result screen

3. Phase 3: Packaging
- PyInstaller build script + internal distribution zip

4. Phase 4: Hardening
- Regression suite, logging cleanup, validation improvements

## Definition of Done for Desktop MVP
1. Non-technical user can run the `.exe` and generate a package without Python installed.
2. Same seed produces same outputs.
3. Different seeds produce varied document presentation.
4. All core scenarios run from UI.
5. Output includes artifacts and manifest files required for QA validation.

## Decommission Note
After this architecture is implemented and reflected in final official docs, delete this temporary file from `docs/`.
