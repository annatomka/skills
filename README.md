# Agent skills

A collection of practical AI/agent skills to improve day-to-day workflows.

These skills are designed for projects that use:
- Next.js
- Playwright
- E2E tests under `__e2e__/`
- a repository-standard command such as `yarn test:e2e-local`

Feel free to copy, adapt, and extend them for your own projects and workflows.

## Usage
Install the skills via `skills`:
```
npx skills@latest add <skill path>
```
For example:

```
npx skills@latest add annatomka/skills/playwright-impact-analysis 
```

## Playwright skills
Skills focused on end-to-end testing workflows using Playwright.

### playwright-impact-analysis
Analyzes local changes and determines which Playwright E2E tests are most likely impacted.

It works in two stages:
1. **Mapping-first**
- Uses `__e2e__/e2e-test-mapping.md` as the primary source of truth when available
2. **Heuristic fallback**
- Uses conservative heuristics only for files that are not covered by the mapping

It can recommend:
- exact test files
- all tests in an E2E folder
- a smoke subset
- the full E2E suite

The output is structured and includes:
- analyzed change scope
- changed files reviewed
- recommended tests grouped by confidence
- exact `yarn test:e2e-local` commands
- assumptions and uncertainty
- candidate mapping improvements when heuristics were needed

Useful for:
- Pull request validation
- Faster local feedback
- CI pipeline optimization
- Large test suites

### playwright-mapping-update
Companion skill for `playwright-impact-analysis`. Use this skill to keep `__e2e__/e2e-test-mapping.md` accurate as the codebase evolves.

It analyzes branch changes and proposes minimal, reviewable mapping updates, such as:
- new mappings for changed source files
- new mappings for added tests
- cleanup of stale mappings after refactors
- path updates for renamed or moved files

The skill is conservative by design:
- it preserves existing structure and conventions
- it proposes only additions, removals, or targeted modifications
- it outputs a diff-style proposal with confidence levels
- it does not apply changes automatically
  
Useful for:
- Keeping mappings current
- Improving recommendation quality
- Scaling E2E coverage in large repositories
---
