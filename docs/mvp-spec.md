# MVP Specification: Deterministic Synthetic Audit Document Generator

## 1. Objective
Deliver a deterministic Python-based generator that creates synthetic, audit-relevant document packages for QA and development testing.

The MVP must support:
- Prompt-driven scenario requests.
- Cross-document linkage across core financial evidence artifacts.
- Ground-truth outputs that allow direct validation against auditor Excel templates.

## 2. In-Scope Document Types (MVP)
1. Invoice
2. Purchase Order (PO)
3. Bill of Lading (BOL)
4. Shipping Document
5. Receipt
6. Credit Memo
7. Bank Statement Entry (line-item level)

Out of scope for MVP (phase 2+):
- OCR simulation noise injection.
- Full trial-balance statement rendering.
- LLM-generated narrative text.

## 3. Required Field Dictionary
All generated records must include values required for extraction and linkage tests.

### 3.1 Common Fields
- document_type
- document_id
- issue_date
- currency
- amount_total
- party_seller_name
- party_buyer_name

### 3.2 Invoice Fields
- invoice_id
- po_id (nullable for negative scenarios)
- invoice_date
- due_date
- line_items[]: description, quantity, unit_price, line_total
- subtotal
- tax_amount
- total_amount

### 3.3 PO Fields
- po_id
- po_date
- vendor_name
- buyer_name
- line_items[]
- po_total

### 3.4 BOL Fields
- bol_id
- po_id
- shipment_date
- carrier_name
- origin
- destination
- shipped_items[]

### 3.5 Shipping Document Fields
- shipment_id
- bol_id
- po_id
- ship_date
- delivery_date
- carrier_name
- status

### 3.6 Receipt Fields
- receipt_id
- invoice_id
- payment_date
- payment_method
- paid_amount
- bank_reference

### 3.7 Credit Memo Fields
- credit_memo_id
- invoice_id
- credit_date
- credit_reason
- credit_amount

### 3.8 Bank Statement Entry Fields
- bank_entry_id
- posting_date
- value_date
- counterparty_name
- reference_text
- debit_credit_flag
- amount
- running_balance
- linked_receipt_id (nullable)

## 4. Linkage Model and Constraints
The generator must enforce a relationship graph for positive scenarios and controlled breaks for negative scenarios.

### 4.1 Primary Link Graph
- PO.po_id -> Invoice.po_id
- PO.po_id -> BOL.po_id
- BOL.bol_id -> ShippingDocument.bol_id
- Invoice.invoice_id -> Receipt.invoice_id
- Receipt.receipt_id -> BankEntry.linked_receipt_id or embedded in reference_text
- CreditMemo.invoice_id -> Invoice.invoice_id

### 4.2 Consistency Rules (Positive Cases)
- Seller and buyer party names stay consistent across linked documents.
- Date ordering is valid: po_date <= shipment_date <= invoice_date <= payment_date <= posting_date.
- Amount continuity is valid within tolerance:
  - Receipt.paid_amount equals Invoice.total_amount minus CreditMemo.credit_amount (if present).
  - Bank entry amount equals Receipt.paid_amount.

### 4.3 Controlled Inconsistency Rules (Negative Cases)
- One linkage key mismatch per scenario where configured.
- Partial references allowed (for example, truncated invoice ID in bank reference text).
- Date anomalies allowed in specific scenario classes.
- Amount differences allowed using bounded deltas.

## 5. Multi-Page Rendering Rules
MVP supports one required multi-page behavior for invoices.

### 5.1 Invoice Continuation
- Page 1 contains invoice header and first N line items.
- Page 2 continues line items and totals.
- Invoice number may be omitted on page 2 when scenario requests continuation behavior.
- Continuation pages must preserve visual and data consistency.

## 6. Scenario Catalog (MVP)
Each scenario emits a package and a ground-truth manifest.

1. Happy path linked set: all relationships valid.
2. Partial match: one weak/partial key linkage (for example short reference text).
3. Key mismatch: exactly one document key mismatch.
4. Amount mismatch: receipt or bank amount differs within configured delta.
5. Date anomaly: one temporal ordering violation.
6. Multi-page invoice continuation: 2-page invoice with continuation semantics.

## 7. Prompt and Request Schema (Generator Input)
Minimal request schema for MVP:

```json
{
  "seed": 20260313,
  "scenario_type": "happy_path|partial_match|key_mismatch|amount_mismatch|date_anomaly|multi_page_invoice",
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
    "enabled": false,
    "target_document": "invoice",
    "page_count": 2,
    "omit_invoice_id_on_continuation": true
  },
  "options": {
    "currency": "USD",
    "max_amount_delta_pct": 5,
    "allow_partial_reference": true
  }
}
```

## 8. Output Package Format
Each run must output a deterministic package folder:

- artifacts/
  - rendered files per document (PDF or JSON template render output in MVP)
- manifest/
  - ground_truth.json
  - linkage_graph.json
  - scenario_metadata.json

### 8.1 ground_truth.json Minimum Content
- Canonical extracted fields per document.
- Expected Excel mapping values:
  - document_id
  - customer_or_vendor_name
  - invoice_amount
  - invoice_date
- Match expectations per relationship edge.

### 8.2 linkage_graph.json Minimum Content
- Node list with document IDs and types.
- Edge list with relation types and expected status:
  - match
  - partial
  - mismatch
  - ambiguous

## 9. Determinism and Reproducibility
- Same input request plus same seed must produce identical structured outputs.
- Random value generation must use a seeded RNG only.
- Generated IDs should be stable for a given seed and scenario index.
- Timestamps and sequence numbers must be deterministic.

## 10. Acceptance Criteria for MVP
1. Generator creates all in-scope document types.
2. Generator supports all six MVP scenario categories.
3. Linked IDs and field consistency pass positive-case validation.
4. Controlled inconsistencies are injected correctly in negative-case scenarios.
5. Multi-page invoice continuation behavior is generated and represented in manifests.
6. Re-running with same seed reproduces equivalent outputs.
7. Ground-truth files are sufficient for automated matching assertions.

## 11. Non-Functional Constraints
- Python implementation with deterministic behavior first.
- No requirement for external LLM service in MVP.
- Keep package and dependency footprint minimal.

## 12. Implementation Order (Recommended)
1. Define internal data models for each document type.
2. Implement deterministic ID/date/amount generators.
3. Implement variation planner (layout, labels, date formats, ID display patterns).
4. Implement linkage engine and constraint validator.
5. Implement scenario injectors by category.
6. Implement renderers and manifest writers.
7. Add regression tests for seed reproducibility, diversity thresholds, and linkage expectations.

## 13. Non-Repetitive Output Strategy (MVP)
To prevent repetitive invoice and supporting evidence templates, the generator must separate:
- Canonical truth values (IDs, dates, amounts, parties, linkage keys).
- Presentation variation values (layout family, labels, date format display, reference text style).

MVP requires deterministic variation selection using the request seed so that:
- Same seed produces identical variation choices.
- Different seeds produce meaningfully different document presentations.

Variation policy details and matrix are defined in:
- docs/variation-strategy.md
