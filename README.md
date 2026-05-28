# Agent skills

A collection of practical AI/agent skills to improve day-to-day workflows.

These skills are designed for projects that use Playwright for end-to-end testing.

They do not assume a specific framework, folder structure, package manager, or E2E command. Each skill should inspect the target repository and prefer its existing test scripts and conventions.

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

Keep Playwright up to date in target repositories when using the diagnosis skills. Newer Playwright versions generate richer `error-context.md` files, which these skills use heavily for investigation.

### playwright-impact-analysis
Analyzes local changes and determines which Playwright E2E tests are most likely impacted.

It works in two stages:
1. **Mapping-first**
- Uses the repository's E2E test mapping file as the primary source of truth when available
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
- exact commands using the repository's detected E2E script or Playwright fallback
- assumptions and uncertainty
- candidate mapping improvements when heuristics were needed

Useful for:
- Pull request validation
- Faster local feedback
- CI pipeline optimization
- Large test suites

### playwright-mapping-update
Companion skill for `playwright-impact-analysis`. Use this skill to keep the repository's E2E test mapping file accurate as the codebase evolves.

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

### playwright-generate
Generates, debugs, and maintains Playwright E2E tests using the target repository's own conventions.

Before writing tests, it discovers:
- testing documentation
- Playwright config
- E2E folders
- package scripts
- nearby specs
- fixtures, helpers, page objects, setup utilities, and data factories

Useful for:
- Adding new E2E coverage
- Updating existing specs
- Validating selectors and flows
- Keeping generated tests consistent with nearby examples

### playwright-single-failure-diagnosis
Diagnoses one failing Playwright test and classifies the likely cause.

It distinguishes between:
- outdated tests
- element overlap or actionability problems
- actual UI/product bugs
- inconclusive cases needing more evidence

Useful for:
- One-off CI failures
- Locator and assertion failures
- Click/actionability issues
- Deciding whether to update a test or fix product code

### playwright-failure-cluster-analysis
Analyzes multiple failing Playwright tests and groups them by likely shared root cause.

It prioritizes cheap evidence first:
- `error-context.md`
- Playwright reports
- screenshots and videos
- source inspection
- selected traces only when needed

Useful for:
- CI failure triage
- Finding common setup or product regressions
- Avoiding opening every trace manually
- Producing compact failure reports

## Media skills
Small utilities for preparing development artifacts.

### mov-to-gif
Converts `.mov` screen recordings or short video clips to optimized `.gif` files with `ffmpeg`.

Requires `ffmpeg` to be installed and available on `PATH`.

It uses palette generation by default for better GIF quality:
```bash
ffmpeg -i <input.mov> -vf "fps=10,scale=840:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -c:v gif <output.gif>
```

Useful for:
- Pull request descriptions
- Bug reports
- Issue reproduction clips
- Documentation snippets
---
