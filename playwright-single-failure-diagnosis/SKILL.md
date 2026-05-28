---
name: playwright-single-failure-diagnosis
description: Use this skill to diagnose one failing Playwright test and decide whether it is failing because the test is outdated, an element is present but blocked/overlapped, or there is an actual UI/product bug. Use static artifacts first, then trace CLI, then playwright-cli and installed playwright-cli skills for live evidence.
---

# Playwright Single Failure Diagnosis

## Purpose

Diagnose one failing Playwright test and classify the failure into one of these outcomes:

1. **Outdated test** — the test expectation, locator, route, copy, fixture, or waiting strategy no longer matches the intended application behavior.
2. **Element present but blocked or overlapped** — the target element exists on the page, but Playwright cannot interact with it because another element, overlay, dialog, cookie banner, sticky header, animation, skeleton, loading layer, or disabled state blocks the action.
3. **Actual UI/product bug** — the application does not render or behave as intended, independent of the test implementation.
4. **Inconclusive** — available evidence is insufficient to classify confidently.

The output should be a short evidence-backed diagnosis, not an immediate code change unless the user asks for a fix.

## When to use this skill

Use this skill when:

- The user provides one failing Playwright test.
- A single failure needs root-cause diagnosis.
- The failure involves a locator timeout, click failure, visibility assertion, text mismatch, URL mismatch, screenshot mismatch, or wrong UI state.
- The user asks whether the test is stale, the UI changed, an element is hidden/covered, or the product is broken.

Do not use this skill for broad clustering of many failures. Use the Playwright failure-cluster analysis skill for bulk triage.

## Available tools and preferred order

### 1. Static artifacts and source inspection

Start here whenever possible.

Inspect:

- The failing test file and failure line.
- Page objects, fixtures, helper functions, and selectors used by the failing test.
- The terminal error output.
- Existing Playwright artifacts, commonly under `test-results/` or the repository's configured output directory.
- Screenshots, videos, traces, error-context files, and report data.
- Recent app/test changes if version-control context is available.

Prefer the repository's E2E script when rerunning tests. Use raw `npx playwright test` only when no repository wrapper exists.

Useful fallback commands:

```bash
<e2e-command> path/to/spec.ts --project=<project> --trace=on
find <playwright-artifacts-dir> -maxdepth 5 -type f | sort
find <playwright-artifacts-dir> -name 'trace.zip' -o -name '*.png' -o -name '*.webm' -o -name '*.txt' | sort
```

Prefer existing artifacts over rerunning the test when the artifact is fresh and complete.

### 2. CLI trace analysis with `npx playwright trace`

Use the trace CLI to inspect the exact failure action and page state without opening the GUI.

Typical workflow:

```bash
TRACE='test-results/path-to-failure/trace.zip'
npx playwright trace open "$TRACE"
npx playwright trace actions --grep='expect|locator|click|fill|goto|wait|hover|check|select'
npx playwright trace action <action-number>
npx playwright trace snapshot <action-number> --name before
npx playwright trace snapshot <action-number> --name after
npx playwright trace close
```

Collect:

- Failing action/assertion.
- Error message and call log.
- Locator or matcher.
- Expected and received values.
- Current URL and page title.
- DOM/accessibility snapshot before and after the failure.
- Last successful action before the failure.
- Visible overlays, dialogs, loading states, error pages, redirects, or empty states.

### 3. `playwright-cli` and installed `playwright-cli` skills

Use `playwright-cli` when live browser inspection can answer questions that static artifacts or traces cannot answer.

The agent may use installed `playwright-cli` skills as local reference guides for command syntax and workflows. Prefer those skills over guessing commands when the workflow is covered by an installed skill.

Relevant installed skills may include:

- Running and Debugging Playwright tests.
- Request mocking.
- Running Playwright code.
- Browser session management.
- Storage state, cookies, and localStorage.
- Test generation.
- Tracing.
- Video recording.
- Inspecting element attributes not visible in snapshots.

Use `playwright-cli` especially when checking whether an element is present but blocked or overlapped.

Typical live debug workflow:

```bash
<e2e-command> path/to/spec.ts --project=<project> --debug=cli
playwright-cli attach <session-name>
playwright-cli snapshot
playwright-cli console error
playwright-cli network --filter='api'
playwright-cli eval '() => ({ href: location.href, title: document.title })'
```

Possible element-state probes:

```bash
playwright-cli snapshot
playwright-cli eval '() => document.elementFromPoint(window.innerWidth / 2, window.innerHeight / 2)?.outerHTML'
playwright-cli eval '() => Array.from(document.querySelectorAll("dialog,[role=dialog],[aria-modal=true],.modal,.overlay,.popover,.toast,.banner")).map(e => ({ tag: e.tagName, text: e.textContent?.slice(0, 120), rect: e.getBoundingClientRect().toJSON?.() ?? e.getBoundingClientRect() }))'
playwright-cli eval '() => ({ active: document.activeElement?.outerHTML, scrollY, viewport: { width: innerWidth, height: innerHeight } })'
```

When a specific selector is known, probe it directly:

```bash
playwright-cli eval '() => {
  const el = document.querySelector("<css-selector>");
  if (!el) return { exists: false };
  const rect = el.getBoundingClientRect();
  const cx = rect.left + rect.width / 2;
  const cy = rect.top + rect.height / 2;
  const top = document.elementFromPoint(cx, cy);
  return {
    exists: true,
    text: el.textContent?.slice(0, 160),
    rect: { x: rect.x, y: rect.y, width: rect.width, height: rect.height },
    visibleByGeometry: rect.width > 0 && rect.height > 0,
    disabled: el.matches(":disabled,[aria-disabled=true]"),
    pointerEvents: getComputedStyle(el).pointerEvents,
    visibility: getComputedStyle(el).visibility,
    display: getComputedStyle(el).display,
    opacity: getComputedStyle(el).opacity,
    topElementIsTarget: top === el || el.contains(top),
    topElement: top?.outerHTML?.slice(0, 300)
  };
}'
```

Prefer accessible locators when diagnosing Playwright locator failures. If the failing test uses `getByRole`, `getByLabel`, `getByText`, or `getByTestId`, compare the intended accessible name or test id to the live page snapshot.

### 4. Playwright MCP

Use Playwright MCP only when an interactive visual exploration is clearly more efficient than command-line inspection.

Use MCP when:

- Visual layering or complex dynamic state is difficult to inspect from snapshots.
- A manual exploratory flow is necessary to understand the failure.
- The UI requires multiple interactions that are awkward through concise CLI commands.

Avoid MCP for routine single-failure diagnosis when trace CLI or `playwright-cli` is enough.

## Diagnosis workflow

### Step 1: Establish the exact failure

Record:

- Test path and title.
- Browser/project.
- Failing line.
- Error message.
- Failing action or assertion.
- Locator, selector, endpoint, or expected value.
- Trace/screenshot/video paths.

Do not classify yet.

### Step 2: Identify the intended behavior

Infer intent from:

- Test name.
- Assertion text.
- Nearby test cases.
- Page object method name.
- Product route or component under test.
- Existing comments or test data setup.

If available, inspect the app/component code around the relevant UI. Determine what the UI should do, not merely what the test expects.

### Step 3: Inspect failure-time page state

Use trace CLI first:

```bash
npx playwright trace actions --grep='expect|locator|click|fill|goto|wait|hover|check|select'
npx playwright trace action <n>
npx playwright trace snapshot <n> --name after
```

Look for:

- Wrong page or route.
- Login/permission redirect.
- App error page.
- Empty state.
- Loading spinner or skeleton.
- Target element absent.
- Target element present with different accessible name/text.
- Target element present but disabled.
- Target element present but underneath another element.
- Modal/dialog/cookie banner/tooltip/popover/sticky header covering it.
- Network or console errors explaining the wrong UI state.

### Step 4: If the element may exist, verify presence versus actionability

Use `playwright-cli snapshot` and, if needed, DOM geometry checks.

Classify the element state:

- **Absent**: not in snapshot or DOM.
- **Present but renamed**: exists with different role/name/text/test id.
- **Present but not visible**: in DOM but hidden via CSS, collapsed, offscreen, opacity, or display.
- **Present but disabled**: disabled attribute or `aria-disabled=true`.
- **Present but covered**: `document.elementFromPoint()` at the target center returns another element.
- **Present but not stable**: animation, transition, layout shift, or loading layer prevents action.

For actionability failures, collect evidence from Playwright call logs and a geometry probe.

### Step 5: Compare test locator against actual page semantics

A test is likely outdated when:

- The intended element exists but the locator no longer matches because the accessible name, role, label, text, or test id changed.
- The UI route changed and the test still expects the old URL.
- Product copy changed but the new copy is correct.
- The component was intentionally redesigned and the old element was replaced by an equivalent new control.
- The test uses brittle CSS/XPath/index-based selectors and the DOM structure changed while user-visible behavior remains correct.
- The expectation asserts an obsolete title, text, count, or ordering.
- The test waits for a state that is no longer part of the intended flow.

Required evidence for this classification:

- The intended behavior is still satisfied by the UI in a different form.
- The failure points to stale locator/expectation details.
- No product error, blocked action, or missing required behavior explains the failure better.

### Step 6: Check for overlap/blocking UI

A failure is likely an element-overlap/actionability issue when:

- The target element exists in the page snapshot or DOM.
- Its bounding box has non-zero size and is near the expected location.
- Playwright call log mentions interception, non-receiving pointer events, instability, or retrying click.
- `document.elementFromPoint()` at the target center returns another element.
- A dialog, overlay, sticky header, cookie banner, toast, tooltip, menu, loading mask, or animation is visible.
- The element becomes actionable after closing the overlay, scrolling, waiting for animation, or dismissing a banner.

Required evidence for this classification:

- Target element presence.
- Blocking element identity.
- Why the blocker is unexpected or how it affects actionability.

Then distinguish:

- **Outdated test with new overlay**: The overlay is expected by current product behavior, and the test should dismiss or handle it.
- **Actual UI bug with overlay**: The overlay should not appear, never disappears, covers critical controls, or prevents intended user interaction.
- **Timing/flake**: The overlay/loading state is transient and the test races it.

### Step 7: Check for actual UI/product bug

A failure is likely an actual UI bug when:

- The expected user-visible behavior is correct and current, but the app does not satisfy it.
- Required UI is absent, broken, disabled, or unreachable.
- The user is on the correct route with correct data, but the component does not render as intended.
- A network/API error, console exception, hydration failure, or app runtime error prevents the UI from working.
- The UI renders an incorrect state, wrong data, wrong permission state, or broken navigation.
- A blocking overlay is not expected and prevents normal user interaction.
- The issue is reproducible manually or with a minimal Playwright interaction independent of the original locator.

Required evidence for this classification:

- The test expectation aligns with intended product behavior.
- The failing condition is visible or observable in page/network/console state.
- The failure is not better explained by stale locators, stale copy, stale test data, or a known intentional UI change.

### Step 8: Handle inconclusive cases honestly

Classify as inconclusive when:

- No trace/screenshot/video is available and rerun is not possible.
- The test intent is unclear.
- The product behavior cannot be inferred from source, nearby tests, or UI state.
- Evidence supports multiple plausible explanations.

Give the smallest next diagnostic command that would resolve the uncertainty.

## Classification decision matrix

| Evidence | Likely classification |
|---|---|
| Element absent, but equivalent new element exists with changed role/name/text | Outdated test |
| Expected text/title/URL differs, and new value appears intentional/current | Outdated test |
| CSS/XPath/index locator fails, accessible/user-facing target still works | Outdated test |
| Target exists, but `elementFromPoint` returns overlay/header/dialog/banner | Element present but blocked/overlapped |
| Target exists, but Playwright call log says another element intercepts pointer events | Element present but blocked/overlapped |
| Target exists but disabled due to app state that should allow action | Actual UI bug |
| Correct route/data, required UI missing with console/API error | Actual UI bug |
| Login/error/empty page appears unexpectedly | Actual UI bug, environment bug, or setup/auth bug; classify based on fixture/network evidence |
| Test data missing or seed failed | Test setup/data issue; not necessarily UI bug |
| Same test passes after longer wait and snapshot shows transient loading | Timing/flakiness; decide whether waiting strategy or app performance is at fault |
| Screenshot changed because intended UI redesign occurred | Outdated visual baseline/test |
| Screenshot changed because UI is broken/misaligned/covered | Actual UI bug or overlap bug |

## Locator-specific heuristics

### `getByRole` failure

Check:

- Did the role change?
- Did the accessible name change?
- Is the element hidden from the accessibility tree?
- Is an equivalent control present under a new name?

Likely outdated test if the equivalent user-facing control exists and works.
Likely UI bug if the intended accessible control is missing or inaccessible.

### `getByText` or `toHaveText` failure

Check:

- Is copy intentionally changed?
- Is localization different?
- Is the old text replaced by semantically equivalent text?
- Is the target component rendering wrong data?

Likely outdated test for intended copy changes.
Likely UI bug for incorrect data or missing required messaging.

### `getByTestId` failure

Check:

- Did `data-testid` change or disappear?
- Is the same visible element present?
- Is there a testing contract around test ids?

Likely outdated test if test ids are not a stable contract and the UI is correct.
Likely product/testability regression if test ids are intentionally part of the contract.

### CSS/XPath failure

Check:

- Did DOM structure change while visible behavior remains correct?
- Is the selector too coupled to layout or component internals?

Usually classify as outdated test unless the visible UI is actually broken.

### Click/actionability failure

Check:

- Playwright call log for actionability retries.
- Whether element is visible, stable, enabled, and receiving events.
- What element is at the click point.
- Whether scroll position or sticky UI changes the click target.

Often classify as overlap/blocking if the target exists but cannot receive pointer events.

## Evidence commands

### Open and inspect trace

```bash
TRACE='test-results/path-to-failure/trace.zip'
npx playwright trace open "$TRACE"
npx playwright trace actions --grep='expect|locator|click|fill|goto|wait|hover|check|select'
npx playwright trace action <n>
npx playwright trace snapshot <n> --name before
npx playwright trace snapshot <n> --name after
npx playwright trace close
```

### Reproduce in CLI debugger

```bash
<e2e-command> path/to/spec.ts --project=<project> --debug=cli
playwright-cli attach <session-name>
playwright-cli snapshot
playwright-cli console error
playwright-cli network --filter='api'
```

### Verify page identity

```bash
playwright-cli eval '() => ({ href: location.href, title: document.title, readyState: document.readyState })'
```

### Check for blocking layers

```bash
playwright-cli eval '() => Array.from(document.querySelectorAll("dialog,[role=dialog],[aria-modal=true],.modal,.overlay,.backdrop,.popover,.toast,.banner,[data-testid*=modal],[data-testid*=overlay]")).map(e => {
  const r = e.getBoundingClientRect();
  const cs = getComputedStyle(e);
  return {
    tag: e.tagName,
    id: e.id,
    className: e.className,
    role: e.getAttribute("role"),
    ariaModal: e.getAttribute("aria-modal"),
    text: e.textContent?.trim().slice(0, 160),
    rect: { x: r.x, y: r.y, width: r.width, height: r.height },
    display: cs.display,
    visibility: cs.visibility,
    opacity: cs.opacity,
    zIndex: cs.zIndex,
    pointerEvents: cs.pointerEvents
  };
})'
```

### Check target actionability by CSS selector

```bash
playwright-cli eval '() => {
  const el = document.querySelector("<css-selector>");
  if (!el) return { exists: false };
  const r = el.getBoundingClientRect();
  const cx = Math.max(0, Math.min(innerWidth - 1, r.left + r.width / 2));
  const cy = Math.max(0, Math.min(innerHeight - 1, r.top + r.height / 2));
  const top = document.elementFromPoint(cx, cy);
  const cs = getComputedStyle(el);
  return {
    exists: true,
    text: el.textContent?.trim().slice(0, 160),
    rect: { x: r.x, y: r.y, width: r.width, height: r.height },
    inViewport: r.bottom > 0 && r.right > 0 && r.top < innerHeight && r.left < innerWidth,
    display: cs.display,
    visibility: cs.visibility,
    opacity: cs.opacity,
    pointerEvents: cs.pointerEvents,
    disabled: el.matches(":disabled,[aria-disabled=true]"),
    topElementIsTarget: top === el || el.contains(top),
    topElement: top?.outerHTML?.slice(0, 400)
  };
}'
```

### Check accessible snapshot

```bash
playwright-cli snapshot
```

Use this to compare the failing locator’s intended role/name/text against what is actually exposed to the accessibility tree.

## Output format

Produce a concise diagnosis report using `output-template.md`.

## Guardrails

- Do not classify as outdated merely because the locator fails.
- Do not classify as a UI bug merely because the test fails.
- Do not classify as overlap unless the target is present and a blocker/actionability issue is evidenced.
- Do not ignore authentication, seed data, permissions, or environment problems.
- Do not rewrite the test before confirming intended product behavior.
- Prefer user-facing locators for any suggested test update.
- Preserve exact artifact paths and failing action details so the diagnosis is reproducible.
