---
name: playwright-mapping-update
description: Update the Playwright E2E test mapping file based on changes in a branch, typically used during PR creation to keep test mappings accurate
---
Update the repository's E2E test mapping file based on local changes.

Your goal is to keep the mapping file accurate with minimal, reviewable changes.

## When to use

This skill should be used:
- when creating a PR
- when new features or tests are added
- when refactoring affects file paths or test coverage

## Workflow

1. Discover the repository's E2E setup
   Inspect the repository before proposing mapping changes:
   - E2E mapping files such as `e2e-test-mapping.md`, `tests/e2e-test-mapping.md`, `e2e/e2e-test-mapping.md`, `__e2e__/e2e-test-mapping.md`, or `<detected-e2e-folder>/e2e-test-mapping.md`
   - Playwright config files such as `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
   - E2E folders such as `e2e/`, `tests/e2e/`, `__e2e__/`, `playwright/`, or folders referenced by Playwright config
   - source/application folders used by the repository

2. Determine change scope
   Analyze local changes compared to the default branch, usually `main` or `master`.

3. Identify relevant changes
   Detect:
   - new or modified source/application files
   - new or modified E2E tests in the detected E2E folders
   - deleted or renamed files

4. Analyze missing mappings

   For each changed source file:
   - check if it exists in the detected E2E mapping file
   - if not → candidate for new mapping

   For each changed test file:
   - check if it is referenced in the mapping
   - if not → candidate for inclusion

5. Infer mappings

   Use:
   - file path similarity
   - feature/domain names
   - existing mapping patterns
   - test file names and describe blocks

   Prefer consistency with existing mapping style.

6. Detect stale mappings

   Identify:
   - mappings pointing to deleted files
   - outdated paths due to refactors

7. Generate minimal updates

   Do NOT rewrite the entire file.

   Only propose:
   - additions
   - removals
   - updates

8. Respect existing structure

   Maintain:
   - section grouping
   - formatting
   - ordering
   - naming conventions

9. Assign confidence

   Use:
   - HIGH → clear direct mapping
   - MEDIUM → strong inference
   - LOW → weak inference (optional suggestion)

10. Output a diff-style proposal

   Show:
   - additions (+)
   - removals (-)
   - modifications (~)

   Do not apply changes automatically.

11. Explain reasoning

   For each proposed change:
   - what changed
   - why mapping is needed or incorrect
   - confidence level

12. Be conservative

   If uncertain:
   - suggest but mark as LOW
   - do not overreach
   - prefer missing mapping over incorrect mapping

Use `heuristics.md` for inference logic.
Use `output-template.md` for formatting.
