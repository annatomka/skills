---
name: playwright-failure-cluster-analysis
description: Analyze multiple failing Playwright tests, group failures by likely shared root cause, preserve exact test titles, and classify clusters when evidence is sufficient. Use error-context.md and static artifacts first; open traces only selectively when extra evidence is needed.
---

# Playwright Failure Cluster Analysis

## Goal

Produce a compact report that groups failing Playwright tests by likely shared cause and, when evidence is strong enough, classifies each cluster as:

- Outdated test issue
- UI/product bug
- Overlap/actionability issue
- Setup/environment/test-data issue
- Timing/flaky issue
- Inconclusive

Do not fix tests unless explicitly asked.

## Evidence order

Use the cheapest useful evidence first:

1. Playwright `error-context.md` files, commonly under `test-results/**/error-context.md` or the repository's configured output directory
2. Playwright JSON/JUnit/blob/HTML report data
3. Screenshots, videos, and other static artifacts
4. Test source, page objects, fixtures, helpers
5. `npx playwright trace` for selected traces only
6. `playwright-cli` and installed `playwright-cli` skills for live inspection only when needed

Do **not** open every `trace.zip`. A trace must be opened only when `error-context.md`, reports, and static artifacts are insufficient to classify or split a cluster.

## Exact test titles are mandatory

Use exact full test titles from reliable sources. Prefer:

1. `error-context.md` → `# Test info` → `Name:` and `Location:`
2. Playwright JSON/JUnit/blob report data
3. Trace metadata from `npx playwright trace open <trace.zip>` → `Title:`
4. HTML report data, if parseable
5. Terminal output, only if it contains the full title chain

Never reconstruct test titles from `test-results` folder names. They may be truncated, escaped, deduplicated, or lossy.

If the exact title cannot be recovered, write `UNKNOWN_EXACT_TITLE` and include the artifact path plus the smallest command needed to recover it.

## Workflow

### 1. Collect failure inventory from `error-context.md`

First scan all error contexts:

```bash
find <playwright-artifacts-dir> -name 'error-context.md' | sort
```

For each `error-context.md`, extract:

- Exact `Name:`
- Exact `Location:`
- Error type and message
- Failing locator/action/assertion
- Playwright call log
- Page snapshot summary
- Artifact folder path

The `Name:` and `Location:` fields are usually enough to identify the exact test without opening the trace.

### 2. Build normalized failure signatures

For each failure, create a compact signature from stable facts:

- Error type: strict mode violation, timeout, assertion mismatch, click intercepted, navigation failure, etc.
- Failing action/assertion: `click`, `fill`, `toBeVisible`, `toHaveText`, `toHaveURL`, etc.
- Locator/selector/role/name/test id
- Expected vs received value
- URL/title/page state if available
- Console/network/API clue if available
- Setup/auth/data clue if available

Normalize noisy values such as timestamps, UUIDs, generated ids, temp paths, ports, and retry suffixes.

### 3. Cluster by shared evidence

Group failures when they share meaningful evidence, for example:

- Same error type and same locator/action
- Same strict-mode ambiguity pattern
- Same expected/received mismatch
- Same missing role/name/test id
- Same wrong route/page title/page snapshot state
- Same API endpoint/status or console error
- Same auth/storage/setup/data fixture problem
- Same visual/component mismatch

Avoid vague clusters like “timeouts” unless the page state and failing locator/action also match.

### 4. Use traces selectively

Open a trace only when it will answer a specific unresolved question, such as:

- Is the page on the wrong route?
- Is the target element present under a different role/name?
- Is an overlay or sticky element blocking the target?
- Did a previous action cause the wrong state?
- Is the failure a cascade from an earlier network/console problem?

Trace workflow for one selected representative failure:

```bash
TRACE='test-results/path-to-failure/trace.zip'
npx playwright trace open "$TRACE"
npx playwright trace actions --grep='expect|locator|click|fill|goto|wait|hover|check|select'
npx playwright trace action <n>
npx playwright trace snapshot <n> --name after
npx playwright trace close
```

Inspect one representative trace per ambiguous cluster first. Open additional traces only to confirm or split the cluster.

### 5. Use `playwright-cli` only for unresolved live-state questions

Use `playwright-cli` when static artifacts and selected traces do not prove the classification.

Useful cases:

- Check the current accessibility snapshot.
- Inspect console/network state.
- Verify storage/auth state.
- Check element attributes not shown in snapshots.
- Confirm whether an element exists but is covered or disabled.

```bash
<e2e-command> path/to/spec.ts --project=<project> --debug=cli
playwright-cli attach <session-name>
playwright-cli snapshot
playwright-cli console error
playwright-cli network --filter='api'
playwright-cli eval '() => ({ href: location.href, title: document.title })'
```

The agent may use installed `playwright-cli` skills as local references for tracing, test debugging, storage state, request mocking, video, and element-attribute inspection.

Prefer the repository's E2E script when rerunning tests. Use raw `npx playwright test` only when no repository wrapper exists.

## Classification rules

### Outdated test issue

Use when the UI appears correct/current but the test is stale.

Evidence examples:

- Locator no longer uniquely identifies the target because the UI now has multiple matching elements.
- Accessible name, role, text, URL, title, test id, or visual baseline changed intentionally.
- Equivalent user-facing behavior exists, but the test points at an old selector or expectation.
- Brittle CSS/XPath/index selector broke after a valid DOM/UI change.

Example: strict mode violation where `getByRole('button', { name: 'Delete' })` now matches both a toolbar delete button and a danger delete button. This is usually an outdated or insufficiently specific locator unless evidence shows the duplicate button is itself unintended.

### UI/product bug

Use when the expected behavior is still correct, but the app is visibly wrong or broken.

Evidence examples:

- Required element is missing on the correct page with correct data.
- Wrong data/state is rendered.
- User flow is blocked unexpectedly.
- App shows runtime, hydration, or console errors.
- API failure creates a broken state users would experience.
- Accessibility regression makes a required control unavailable.

### Overlap/actionability issue

Use when the target exists but cannot be interacted with.

Evidence examples:

- Call log says another element intercepts pointer events.
- Snapshot shows modal, banner, toast, sticky header, menu, tooltip, or loading mask covering the target.
- `document.elementFromPoint()` proves another element is on top.
- Target is visible but disabled, unstable, offscreen, or not receiving events.

### Setup/environment/test-data issue

Use when the UI failure is likely caused by broken preconditions.

Evidence examples:

- Login/storage state expired.
- Seed data missing or duplicated.
- Wrong base URL, config, feature flag, or project.
- Backend/test environment unavailable.
- Shared fixture/global setup failed.

### Timing/flaky issue

Use when the UI likely becomes correct but the test races it.

Evidence examples:

- Retry passes.
- Snapshot shows transient loading/skeleton/animation.
- Timeout occurs before a stable user-visible condition.
- Failure disappears with a targeted wait for app-ready state.

### Inconclusive

Use when evidence is insufficient or multiple explanations remain plausible.

Do not force classification. Provide the smallest next command or artifact needed.

## Confidence rules

- **High**: Same signature and same page/error evidence across multiple failures.
- **Medium**: Same signature and plausible shared cause, with at least one representative artifact confirming it.
- **Low**: Similar symptoms but weak, partial, or mixed evidence.

Only classify as **Outdated test issue** or **UI/product bug** at Medium or High confidence. Otherwise mark Inconclusive.

## Compact report format

Use `output-template.md` for response formatting.

## Final checks

Before responding:

- Prefer `error-context.md` over opening traces.
- Do not open every trace.
- Every listed test has exact `Name:` and `Location:` or `UNKNOWN_EXACT_TITLE`.
- Clusters are evidence-based, not folder-name-based.
- Classification is only asserted when confidence is sufficient.
- The report is compact and avoids long trace dumps or irrelevant logs.
