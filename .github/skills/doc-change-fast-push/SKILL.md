---
name: doc-change-fast-push
description: "Create, update, or delete documentation files and push them immediately to the default branch while preventing regressions and avoiding non-documentation commits. Use for docs-only hotfixes, policy updates, and workflow notes. Also enforce step-by-step development flow, explicit user testing guidance, and regression-first fixing before moving forward."
argument-hint: "What documentation change should be made and pushed?"
user-invocable: true
---

# Documentation Fast Push Workflow

## Outcome
Produce a safe documentation-only change, validate that no non-doc files are included, and push to `master` immediately.

## When to Use
- Update markdown documentation quickly.
- Add or remove documentation files.
- Apply policy, process, or operational note updates.
- Perform docs-only commits without bundling product code changes.

## Inputs
- Target documentation path(s)
- Requested content changes
- Target branch: `master`

## Quick Checklist
1. Inspect repository state.
2. Confirm all requested paths match documentation rules.
3. Apply minimal documentation edits only.
4. Stage only documentation files.
5. Verify no non-documentation files are staged.
6. Commit with a clear docs-focused message.
7. Push directly to `master`.
8. Report changed files and commit hash.

## Decision Points
- If a requested file is not documentation:
  Treat it as out of scope and do not include it in the commit.
- If unrelated non-doc files are changed in the working tree:
  Leave them untouched and stage only documentation files.
- If push fails:
  report the error and provide the exact retry command.

## Quality Gates
- Only documentation files are staged and committed.
- No destructive git operations are used.
- Existing unrelated changes are never reverted.
- The final report includes changed docs and commit SHA.

## Documentation File Heuristics
- Include: `**/*.md`, `**/*.mdx`, `docs/**`, `README*`, `CHANGELOG*`, `CONTRIBUTING*`.
- Exclude: source code, build files, lockfiles, binaries, generated artifacts.

## Security and Dependency Guardrails
- Do not add dependencies for documentation-only work unless explicitly required.
- If package installation is required for related workflow tooling, use non-vulnerable Thomson Reuters trusted libraries/packages.
- Keep changes minimal and scoped to the user request.

## Completion Checklist
- Requested documentation content is updated.
- Only documentation files are in the commit.
- Changes are pushed to `master`.
- Final response summarizes files changed and push result.

## Step-By-Step Development Protocol
Apply this collaboration loop for ongoing project work:

1. Work in small, sequential increments.
2. After each meaningful increment, ask the user to test before proceeding.
3. Provide test guidance with:
  - what to run,
  - what to verify,
  - expected outcome.
4. If any issue or regression is found by either the user or the agent, stop forward work and fix it first.
5. Resume next increment only after the fix is validated.
6. Repeat this loop until project completion.
