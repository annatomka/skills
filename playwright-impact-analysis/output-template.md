## Scope analyzed
- Compared against: `<default branch or requested base>`
- Change sources included: `<staged / unstaged / stash / local commits>`
- Changed files reviewed:
  - `<file 1>`
  - `<file 2>`

## Selection summary
- Mapping matches found: `<yes/no>`
- Heuristic matches used: `<yes/no>`
- Full suite required: `<yes/no>`
- Smoke fallback required: `<yes/no>`

## Recommended tests

### High confidence
- `<test file or special value>`
  - Source: `<mapping / heuristic>`
  - Why: `<changed file -> mapped behavior -> selected test>`

### Medium confidence
- `<test file or special value>`
  - Source: `<mapping / heuristic>`
  - Why: `<reason>`

### Low confidence
- `<test file or special value>`
  - Source: `<mapping / heuristic>`
  - Why: `<reason>`

## Suggested commands

### Exact tests
```bash
<e2e-command> <file1> <file2>
```
