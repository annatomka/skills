# Heuristics for Unmapped Files

If the project contains an E2E test mapping file, use that file first.
Only use these heuristics when the mapping file is missing or does not cover a changed file.

The mapping file is the primary source of truth.

## 1. General principle

Be conservative.
Prefer a slightly broader safe recommendation over a narrow but weak guess.

If confidence is low:
- recommend `SMOKE`
- or a broader relevant area if there is clear domain evidence

## 2. File-type guidance

### Routes / pages
Examples:
- `src/pages/**/*.tsx`
- `src/app/**/page.tsx`
- `app/routes/**`
- `routes/**`
- other framework route or page entry files

Likely impact:
- page entry
- navigation
- route access
- forms and page-specific flows

Recommendation:
- look for matching E2E specs by route name, page name, or domain term
- if unclear, recommend `SMOKE`

### Components
Examples:
- `src/components/**`
- `app/components/**`
- `components/**`
- `packages/ui/**`

Likely impact:
- UI interaction flows
- modals
- tables
- forms
- domain-specific user journeys

Recommendation:
- infer feature/domain from parent folder name
- look for matching E2E spec names or titles
- if component appears shared or generic, broaden toward `SMOKE`

### Hooks
Examples:
- `src/hooks/**`
- `hooks/**`
- `composables/**`

Likely impact:
- cross-cutting behavior
- multiple flows
- indirect regressions

Recommendation:
- infer domain from hook name
- if hook is feature-specific, recommend that feature’s likely E2E specs
- if hook is shared/global, recommend `SMOKE` or broader coverage

### Services / API / config
Examples:
- `src/services/**`
- `src/pages/api/**`
- `app/api/**`
- `server/**`
- `lib/api/**`
- config files
- app bootstrap

Likely impact:
- broad regressions
- data flow changes
- environment-dependent behavior

Recommendation:
- if central/shared, broaden significantly
- use full suite for obviously global changes
- otherwise recommend `SMOKE` plus likely domain tests

### Styles
Examples:
- `*.scss`
- `*.css`

Likely impact:
- layout and visual rendering

Recommendation:
- use `SMOKE` unless there is a known layout-critical feature area

## 3. Naming-based matching

When a file is unmapped, try matching on:
- folder names
- feature names
- route names
- key nouns in the file path

Examples:
- `src/components/Billing/...` → billing-related specs
- `src/pages/projects/...` → project-related specs
- `src/hooks/usePermissions.ts` → auth/access-related specs
- `app/routes/account/...` → account-related specs
- `packages/ui/Dialog/...` → dialog or shared UI specs

## 4. Risk multipliers

Broaden the recommendation when changes touch:
- auth
- permissions
- routing
- shared hooks
- API/service layer
- app bootstrap
- configuration
- websocket/realtime behavior
- editor/canvas behavior with many integration points

## 5. Confidence assignment for heuristic results

Use:
- HIGH only when the connection is very direct
- MEDIUM when the domain match is clear but indirect
- LOW when the relationship is weak and the recommendation is mainly precautionary

Heuristic results should usually be MEDIUM or LOW unless the signal is unusually strong.

## 6. Mapping improvement suggestions

When heuristics produce a strong, repeated pattern, suggest a candidate mapping addition in this format:

- `<source-path>` → `<e2e-folder>/target.spec.ts`
- `<source-folder>/**` → `<e2e-folder>/folder/**`

Only suggest additions when confidence is reasonably strong.
