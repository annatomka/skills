---
name: playwright-impact-analysis
description: Use this skill when user asks for analyzing changes. Determine which Playwright e2e tests should run for local changes by using the E2E test mapping file first, then falling back to safe heuristics for unmapped files
---
Determine the smallest safe subset of Playwright e2e tests affected by local changes.

This skill is repository-aware. Do not assume a specific framework, E2E folder, package manager, or test command.

Your primary source of truth is the repository's E2E test mapping file, if one exists.

Common mapping locations include:
- `e2e-test-mapping.md`
- `tests/e2e-test-mapping.md`
- `e2e/e2e-test-mapping.md`
- `__e2e__/e2e-test-mapping.md`
- `<detected-e2e-folder>/e2e-test-mapping.md`

When this file is present:
- read it from the current repository
- use it as the primary source of truth for test selection
- apply its precedence, confidence, special values, and fallback rules exactly

When this file is not present:
- fall back to the heuristics in this skill
- clearly state that no project mapping file was found

Your goal is to avoid running the full E2E suite unless necessary, while minimizing the chance of missing affected tests.

## Workflow

1. Discover the repository's E2E setup
   Inspect the repository before recommending commands or paths:
   - `package.json` scripts containing `playwright`, `e2e`, `test:e2e`, or similar names
   - Playwright config files such as `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
   - common E2E folders such as `e2e/`, `tests/e2e/`, `__e2e__/`, `playwright/`, or folders referenced by Playwright config
   - E2E mapping files in common locations

   Prefer repository scripts over raw Playwright commands because they may apply environment setup, filtering, retries, or project-specific exclusions.

2. Determine the change scope
   Identify what local changes should be analyzed:
   - unstaged changes
   - staged changes
   - stash, if the user explicitly asks
   - local commits different from the default branch

   Default to all local changes compared to the default branch, usually `main` or `master`.

3. Collect changed files
   Build the list of changed files for the selected scope.

4. Use the mapping file first
   Read the detected E2E mapping file and resolve each changed file against the mapping.

   Apply the mapping rules exactly as defined there:
   - special values
   - pattern rules
   - match precedence
   - confidence merge rules
   - selection procedure
   - fallback rules

5. Prefer explicit mappings over heuristics
   If a changed file has an explicit mapping, use that mapping.
   Only use heuristics when:
   - no mapping matches
   - a mapping is too broad and additional explanation is useful
   - the user asks for deeper reasoning

6. Resolve special values
   Handle these exactly:
   - `ALL` → recommend running the full E2E suite with the detected E2E command
   - `SMOKE` → recommend the repository's smoke command or append the appropriate Playwright grep flag when safe
   - `<e2e-folder>/**` → include all tests in that folder
   - exact test files → include those exact files

7. Merge and deduplicate results
   If the same test is selected multiple times:
   - keep it only once
   - keep the highest confidence level

   Confidence priority:
   - HIGH
   - MEDIUM
   - LOW

8. Use heuristics only for unmapped files
   For files not covered by the mapping:
   - apply the guidance from `heuristics.md`
   - be conservative
   - prefer `SMOKE` over guessing exact tests unless confidence is strong

9. Produce actionable output
   Output:
   - the scope analyzed
   - changed files reviewed
   - recommended tests grouped by confidence
   - whether `ALL` or `SMOKE` was triggered
   - exact commands using the detected repository E2E command
   - assumptions and uncertainty

10. Do not overclaim
   Never claim the selected tests are guaranteed sufficient unless the mapping is explicit and direct.
   Clearly distinguish:
   - mapping-based selections
   - heuristic selections
   - fallbacks

11. Suggest mapping improvements when useful
   If you had to rely on heuristics and the inferred relationship is strong, suggest one or more candidate additions to the detected E2E mapping file.

## Command rules

Always prefer the repository-standard E2E command:
- full suite: `<e2e-command>`
- smoke: `<e2e-command> --grep "@smoke"`, unless the repository has a dedicated smoke script
- exact files: `<e2e-command> <file1> <file2>`
- grep by test name only if exact files are unavailable and there is a strong reason

If a custom script exists, use it. Examples include `yarn test:e2e-local`, `npm run e2e`, `pnpm test:e2e`, or another project-specific wrapper.

Use `npx playwright test` only as a fallback when no repository script is available.

Use `output-template.md` for response formatting.
Use `heuristics.md` only when the mapping file does not fully answer the request.
