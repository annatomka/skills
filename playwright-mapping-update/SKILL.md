---
name: playwright-mapping-update
description: Update the Playwright E2E test mapping file based on changes in a branch, typically used during PR creation to keep test mappings accurate
---
Update the E2E test mapping file (`__e2e__/e2e-test-mapping.md`) based on local changes.

Your goal is to keep the mapping file accurate with minimal, reviewable changes.

## When to use

This skill should be used:
- when creating a PR
- when new features or tests are added
- when refactoring affects file paths or test coverage

## Workflow

1. Determine change scope
   Analyze local changes compared to `main`.

2. Identify relevant changes
   Detect:
   - new or modified source files (`src/**`)
   - new or modified e2e tests (`__e2e__/**`)
   - deleted or renamed files

3. Analyze missing mappings

   For each changed source file:
   - check if it exists in `__e2e__/e2e-test-mapping.md`
   - if not → candidate for new mapping

   For each changed test file:
   - check if it is referenced in the mapping
   - if not → candidate for inclusion

4. Infer mappings

   Use:
   - file path similarity
   - feature/domain names
   - existing mapping patterns
   - test file names and describe blocks

   Prefer consistency with existing mapping style.

5. Detect stale mappings

   Identify:
   - mappings pointing to deleted files
   - outdated paths due to refactors

6. Generate minimal updates

   Do NOT rewrite the entire file.

   Only propose:
   - additions
   - removals
   - updates

7. Respect existing structure

   Maintain:
   - section grouping
   - formatting
   - ordering
   - naming conventions

8. Assign confidence

   Use:
   - HIGH → clear direct mapping
   - MEDIUM → strong inference
   - LOW → weak inference (optional suggestion)

9. Output a diff-style proposal

   Show:
   - additions (+)
   - removals (-)
   - modifications (~)

   Do not apply changes automatically.

10. Explain reasoning

   For each proposed change:
   - what changed
   - why mapping is needed or incorrect
   - confidence level

11. Be conservative

   If uncertain:
   - suggest but mark as LOW
   - do not overreach
   - prefer missing mapping over incorrect mapping

Use `heuristics.md` for inference logic.
Use `output-template.md` for formatting.