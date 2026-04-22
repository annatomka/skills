---
name: playwright-impact-analysis
description: Use this skill when user asks for analyzing changes. Determine which Playwright e2e tests should run for local changes by using the E2E test mapping file first, then falling back to safe heuristics for unmapped files
---
Determine the smallest safe subset of Playwright e2e tests affected by local changes.

This project uses:
- Next.js
- Playwright
- E2E tests under `__e2e__/`
- The repository-standard command `yarn test:e2e-local`

Your primary source of truth is the project file `__e2e__/e2e-test-mapping.md`, if it exists.

When this file is present:
- read it from the current repository
- use it as the primary source of truth for test selection
- apply its precedence, confidence, special values, and fallback rules exactly

When this file is not present:
- fall back to the heuristics in this skill
- clearly state that no project mapping file was found

Your goal is to avoid running the full E2E suite unless necessary, while minimizing the chance of missing affected tests.

## Workflow

1. Determine the change scope
   Identify what local changes should be analyzed:
   - unstaged changes
   - staged changes
   - stash, if the user explicitly asks
   - local commits different from `main`

   Default to all local changes compared to `main`.

2. Collect changed files
   Build the list of changed files for the selected scope.

3. Use the mapping file first
   Read `e2e-test-mapping.md` and resolve each changed file against the mapping.

   Apply the mapping rules exactly as defined there:
   - special values
   - pattern rules
   - match precedence
   - confidence merge rules
   - selection procedure
   - fallback rules

4. Prefer explicit mappings over heuristics
   If a changed file has an explicit mapping, use that mapping.
   Only use heuristics when:
   - no mapping matches
   - a mapping is too broad and additional explanation is useful
   - the user asks for deeper reasoning

5. Resolve special values
   Handle these exactly:
   - `ALL` → recommend running the full E2E suite with `yarn test:e2e-local`
   - `SMOKE` → recommend running `yarn test:e2e-local --grep "@smoke"`
   - `__e2e__/folder/**` → include all tests in that folder
   - exact test files → include those exact files

6. Merge and deduplicate results
   If the same test is selected multiple times:
   - keep it only once
   - keep the highest confidence level

   Confidence priority:
   - HIGH
   - MEDIUM
   - LOW

7. Use heuristics only for unmapped files
   For files not covered by the mapping:
   - apply the guidance from `heuristics.md`
   - be conservative
   - prefer `SMOKE` over guessing exact tests unless confidence is strong

8. Produce actionable output
   Output:
   - the scope analyzed
   - changed files reviewed
   - recommended tests grouped by confidence
   - whether `ALL` or `SMOKE` was triggered
   - exact `yarn test:e2e-local` commands
   - assumptions and uncertainty

9. Do not overclaim
   Never claim the selected tests are guaranteed sufficient unless the mapping is explicit and direct.
   Clearly distinguish:
   - mapping-based selections
   - heuristic selections
   - fallbacks

10. Suggest mapping improvements when useful
   If you had to rely on heuristics and the inferred relationship is strong, suggest one or more candidate additions to `e2e-test-mapping.md`.

## Command rules

Always use the repository-standard command:
- full suite: `yarn test:e2e-local`
- smoke: `yarn test:e2e-local --grep "@smoke"`
- exact files: `yarn test:e2e-local <file1> <file2>`
- grep by test name only if exact files are unavailable and there is a strong reason

Do not default to `npx playwright test`.

Use `output-template.md` for response formatting.
Use `heuristics.md` only when the mapping file does not fully answer the request.