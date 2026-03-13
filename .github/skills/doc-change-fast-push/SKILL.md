---
name: doc-change-fast-push
description: "Create, update, or delete documentation files and push them immediately to the default branch while preventing regressions and avoiding non-documentation commits. Use for docs-only hotfixes, policy updates, and workflow notes."
argument-hint: "What documentation change should be made and pushed?"
user-invocable: true
---

# Documentation Fast Push Workflow

## Outcome
Produce a safe documentation-only change, validate that no non-doc files are included, and push to the repository default branch immediately.

## When to Use
- Update markdown documentation quickly.
- Add or remove documentation files.
- Apply policy, process, or operational note updates.
- Perform docs-only commits without bundling product code changes.

## Inputs
- Target documentation path(s)
- Requested content changes
- Repository default branch name (auto-detect if not provided)

## Procedure
1. Inspect repository state.
2. Identify whether requested paths are documentation files.
3. Apply minimal documentation edits only.
4. Verify no non-documentation files are staged.
5. Run lightweight checks to avoid regressions caused by accidental edits.
6. Commit documentation files with a clear message.
7. Push directly to the default branch.
8. Report changed files and commit hash.

## Decision Points
- If a requested file is not documentation:
  Treat it as out of scope and do not include it in the commit.
- If unrelated non-doc files are changed in the working tree:
  Leave them untouched and stage only documentation files.
- If default branch cannot be detected:
  Fall back to `main`, then `master`, and report what was used.
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
- Changes are pushed to the default branch.
- Final response summarizes files changed and push result.
