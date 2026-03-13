# Session Handoff - Full Context - 2026-03-13

## Purpose of This Handoff
This document captures the complete discussion, decisions, repository operations, and agreed next steps so a new chat session can continue without losing context.

## Original User Goal
Build a project for Audit Intelligence testing where users can generate synthetic bank statements and supporting evidence documents for QA and development validation.

The generated artifacts should support audit-trail matching against auditor-provided Excel templates with columns such as:
- Document ID
- Customer or Vendor Name
- Invoice Amount
- Invoice Date

The central need is to reduce manual effort in creating realistic and edge-case-heavy test documents.

## Problem Statement Refinement Agreed in Conversation
The problem is not generic document generation. It is generation of linked, test-ready evidence packs for audit validation.

Required generation behavior discussed:
- Single or multi-page invoices
- Split-page invoice continuation behavior (for example, totals or dates rolling forward on page 2)
- Cross-document linkage generation across invoice, PO, BOL, shipping docs, receipts, credit memos, trial-balance outcomes, and bank statements
- Scenarios that test matching, non-matching, and ambiguous relationships

## Architecture Direction Discussed
Two options were evaluated:

1. AI plus rendering pipeline
- AI for prompt interpretation and scenario creation
- Deterministic rendering for final PDF or image output

2. Deterministic Python-only generation
- Rule-based scenario definitions
- Template-driven rendering
- Seeded reproducibility

Final direction chosen for now:
- Start with deterministic Python generation
- Keep LLM optional for later

Reasoning:
- Better repeatability for QA regression testing
- Lower security and operations complexity
- High control over linkage and edge-case construction

## GitHub Pages and Secrets Clarification
Discussion outcome:
- GitHub Pages can host static UI only
- GitHub Pages cannot safely hold runtime API keys
- GitHub Secrets are for CI and workflows, not browser-side runtime use
- If LLM is added later, key usage must stay in a backend service

## Repository Naming Decision
User requested removal of "AI" from the repository name.

Completed remote rename:
- Old: JayLathiaTR/ai-auto-bank-and-supporting-statement-generator
- New: JayLathiaTR/auto-document-generator

Local `origin` remote was updated to:
- https://github.com/JayLathiaTR/auto-document-generator.git

## Local Folder Rename Status
Attempted to rename local folder in place.

Result:
- Rename blocked by active process file lock on Windows

Workaround completed:
- Cloned the renamed repository into a new local folder:
  - C:/GitHub/AA_POC_Projects/auto-document-generator

Current state:
- Old folder still exists:
  - C:/GitHub/AA_POC_Projects/ai-auto-bank-and-supporting-statement-generator
- New folder exists and is usable:
  - C:/GitHub/AA_POC_Projects/auto-document-generator

## Documentation Files Created During Session
1. docs/problem-statement.md
2. docs/session-handoff-2026-03-13.md

## Commits and Pushes Completed During Session
1. 5ae72ab docs(skill): add documentation fast push workflow
2. e02d03f docs(skill): refine checklist and branch policy
3. 13c8278 docs: add audit intelligence problem statement
4. 01224b6 docs: add session handoff summary

All above were pushed to `master`.

## Skill Created During Session
A workspace skill was created and refined:
- .github/skills/doc-change-fast-push/SKILL.md

Behavior of that skill:
- Documentation-only change workflow
- Immediate push pattern
- Guardrails to avoid committing non-doc files

## Security and Workflow Constraints Discussed
- Keep documentation pushes docs-only
- Avoid regressions
- Prefer deterministic generation first
- Use trusted packages when dependencies are needed

## Agreed Product Direction Going Forward
Build a deterministic synthetic document generator in Python that can scale test scenario breadth without requiring an LLM.

Target outcomes:
- Large QA-ready scenario catalog
- Linked document evidence chains
- Realistic but synthetic data
- Repeatable generation for regression

## Recommended Next Session Starting Point
Start in:
- C:/GitHub/AA_POC_Projects/auto-document-generator

Then execute:
1. Draft `docs/mvp-spec.md` with exact document types and required fields.
2. Define scenario schema for generation requests.
3. Define linkage model rules for invoice, PO, BOL, receipt, credit memo, bank entry.
4. Define edge-case catalog by category.
5. Scaffold Python project structure for generator engine and renderers.

## Suggested MVP Spec Sections
1. In-scope document types
2. Required field dictionary per document type
3. Linkage constraints and relationship graph
4. Multi-page rendering rules
5. Positive and negative test scenario definitions
6. Output package format and ground-truth manifest
7. Deterministic seed and reproducibility strategy

## Notes for Continuity
If chat history is lost after switching folders, this document plus `docs/problem-statement.md` should be treated as source-of-truth context for resuming work.
