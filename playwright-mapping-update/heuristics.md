# Mapping Update Heuristics

## 1. Source → Test inference

Map based on:
- feature name similarity
- folder structure
- route or page names, when the framework has file-based routing
- domain terms

Examples:
- `src/pages/projects/**` → `<e2e-folder>/projects.spec.ts`
- `app/routes/account/**` → `<e2e-folder>/account.spec.ts`
- `src/components/Diagram/**` → `<e2e-folder>/diagram.spec.ts`
- `packages/ui/Dialog/**` → `<e2e-folder>/dialog.spec.ts`

---

## 2. Test → Source inference

When a new test is added:
- infer what feature it covers
- match against existing mapping patterns

Example:
- `diagram-advanced.spec.ts` → diagram-related hooks/components

---

## 3. Pattern reuse

Prefer existing patterns:
- if similar mappings exist, reuse structure
- keep consistency across sections

---

## 4. Confidence rules

HIGH:
- exact feature match
- strong naming alignment

MEDIUM:
- indirect relationship
- shared domain

LOW:
- guess based on weak signals

---

## 5. Refactor handling

Detect:
- renamed files
- moved folders

Update mappings accordingly instead of duplicating entries.

---

## 6. Avoid over-mapping

Do NOT:
- map generic utilities aggressively
- assign ALL unless clearly necessary
- create noisy mappings

---

## 7. Section placement

Place new mappings in the most relevant section:
- Projects
- Diagram
- Auth
- etc.

If no section fits, suggest creating a new one.
