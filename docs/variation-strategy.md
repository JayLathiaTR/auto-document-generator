# Variation Strategy Matrix (MVP)

## Objective
Prevent repetitive output templates while preserving deterministic generation and linkage correctness.

This strategy defines controlled variability across layout, language, identifiers, and business logic while maintaining audit-test reliability.

## Design Principle
Use two layers:
1. Core truth layer: canonical values for IDs, parties, dates, and amounts used for linkage and validation.
2. Presentation layer: varied rendering patterns that display the same core truth in different ways.

Only the presentation layer changes for style diversity. Core truth remains stable unless a scenario explicitly injects mismatches.

## Variation Dimensions and Controls

| Dimension | Examples | Applies To | Control Key | Deterministic Rule |
|---|---|---|---|---|
| Layout family | Compact, tabular, header-heavy, continuation-first | Invoice, PO, BOL, shipping, receipt | layout_family | Pick from weighted pool via seed |
| Typography and spacing | Font size, row spacing, alignment choices | All rendered documents | style_profile | Deterministic choice per document index |
| Field labels | Invoice No vs Invoice ID, Vendor vs Supplier | Invoice, PO, receipt | label_synonym_set | Use synonym map keyed by locale/profile |
| Date format | YYYY-MM-DD, DD/MM/YYYY, MMM DD, YYYY | All docs | date_format_profile | Selected once per package unless overridden |
| ID formatting | INV-2026-0001 vs INV/26/0001 | Invoice, PO, BOL, receipt refs | id_pattern_profile | Pattern chosen from seeded set |
| Currency display | 1234.50, 1,234.50, USD 1,234.50 | Invoice, receipt, bank entry | amount_format_profile | Format only, no value changes |
| Line item shape | 3 to 20 line items, grouped categories | Invoice, PO, BOL | line_item_profile | Bounded by scenario config |
| Continuation behavior | Header repeat on page 2 true/false | Multi-page invoice | continuation_profile | Fixed by scenario and seed |
| Reference text style | Receipt ID first, invoice ID first, mixed tokens | Receipt, bank entry | reference_text_profile | Generated from grammar templates |
| Optional fields | Payment terms, ship method, notes | Invoice, shipping, receipt | optional_field_profile | Include based on deterministic threshold |

## Required Guardrails
1. Never vary linkage keys beyond allowed mismatch scenarios.
2. Never vary numeric truth values when variation intent is formatting-only.
3. Keep one intentional inconsistency type per negative scenario unless explicitly configured.
4. Maintain valid chronology in non-date-anomaly scenarios.
5. Preserve party identity consistency in positive scenarios.

## Scenario-Level Diversity Targets
For each batch request, enforce minimum diversity:

| Batch Size | Minimum Distinct Layout Families | Minimum Distinct Label Sets | Minimum Distinct Date Formats |
|---|---|---|---|
| 1 to 5 packages | 2 | 2 | 2 |
| 6 to 20 packages | 3 | 3 | 3 |
| 21+ packages | 4 | 4 | 4 |

If deterministic selection under-separates outputs, rebalance using secondary deterministic shuffling derived from seed and package index.

## Generator Wiring (Python)

## Component Roles
1. SeedManager
- Derives deterministic sub-seeds per package and per document.

2. ScenarioBuilder
- Builds core truth entities and linkage graph.
- Applies scenario mismatch injectors when needed.

3. VariationPlanner
- Chooses variation profiles from weighted pools.
- Produces a variation_plan.json artifact for transparency.

4. Renderer
- Applies style and label variations using variation plan.
- Emits document artifacts without changing canonical truth.

5. ManifestWriter
- Writes ground_truth.json, linkage_graph.json, scenario_metadata.json, variation_plan.json.

## Suggested Data Contracts

```json
{
  "variation_plan": {
    "package_id": "PKG-20260313-0001",
    "layout_family": "tabular_dense",
    "label_synonym_set": "set_b",
    "date_format_profile": "dd_mm_yyyy",
    "id_pattern_profile": "slash_year_short",
    "amount_format_profile": "usd_prefixed",
    "reference_text_profile": "receipt_first",
    "optional_field_profile": "medium"
  }
}
```

## Deterministic Selection Pattern
Use deterministic sub-seeding:
- package_seed = hash(global_seed, package_index)
- document_seed = hash(package_seed, document_type, document_index)

Use package_seed for variation profile selection and document_seed for local rendering details.

## Validation Rules for Non-Repetitive Output
After generation, run diversity checks:
1. Template repetition ratio threshold: no single layout family should exceed 60 percent within a batch larger than 10.
2. Label-set repetition threshold: at least 2 label sets in batches larger than 5.
3. Date format diversity threshold: at least 2 formats in batches larger than 5.
4. Canonical truth integrity: linkage and amounts must still satisfy scenario constraints.

## Recommended Default Weights (MVP)

| Profile Type | Option | Weight |
|---|---|---|
| layout_family | compact | 0.30 |
| layout_family | tabular_dense | 0.30 |
| layout_family | header_heavy | 0.25 |
| layout_family | continuation_first | 0.15 |
| label_synonym_set | set_a | 0.40 |
| label_synonym_set | set_b | 0.35 |
| label_synonym_set | set_c | 0.25 |

Weights can be tuned later, but they must stay explicit and versioned in configuration.

## Configuration Placement
Store variation definitions in data files:
- config/variation/layout_families.json
- config/variation/label_sets.json
- config/variation/date_formats.json
- config/variation/weights.json

This keeps variation behavior editable without code changes.

## MVP Acceptance Additions
1. Two runs with different seeds produce visibly different style combinations.
2. Two runs with same seed produce identical variation_plan.json and equivalent artifacts.
3. Diversity thresholds pass for batch runs.
4. Linkage integrity remains valid across all style variations.

## Example Request Extensions

```json
{
  "seed": 20260313,
  "batch_size": 12,
  "diversity_policy": {
    "enforce_min_layout_families": 3,
    "enforce_min_label_sets": 3,
    "max_single_layout_ratio": 0.6
  },
  "variation_overrides": {
    "allow_date_formats": ["yyyy_mm_dd", "dd_mm_yyyy", "mon_dd_yyyy"],
    "allow_layout_families": ["compact", "tabular_dense", "header_heavy"]
  }
}
```

## Team Usage Guidance
- QA can request broad diversity by increasing batch_size and setting diversity_policy.
- Developers can pin a seed for bug reproduction.
- Audit analysts can compare extraction behavior across style variants using the same core truth scenario.
