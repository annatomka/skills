---
name: tldr
description: Shrink Claude's previous answer (or every finding so far in the session) into a one-sentence verdict plus up to three short findings. Use only when the user types /tldr or asks for "the short version", "one-sentence verdict", "summarize what you found".
argument-hint: [all] (optional: summarize every finding in the session, not only the last answer)
disable-model-invocation: true
---

# TL;DR

Turn a long answer into something that can be read in ten seconds.

## What to summarize

- No argument: your previous answer in this conversation.
- `all`: every finding and conclusion so far in this session, with later corrections replacing the
  earlier claims they correct.

## Output

Exactly this shape, nothing before or after it:

```
**Verdict:** <one sentence that answers the user's question: yes / no / partly, and why>

- <finding 1>
- <finding 2>
- <finding 3>

**Next:** <the one thing to do, or "nothing">
```

- The verdict is one sentence, at most about 25 words. Lead with the answer ("No, ...", "Yes, ...",
  "Partly: ...").
- At most three findings, one line each, at most about 20 words each. Pick the ones that change what
  the user does. Fewer is fine.
- A finding may end with one pointer (a file:line, test name or link) only when it is the main
  evidence.
- "Next" is one action at most. Write "nothing" when there is no action.

## Rules

- Summarize only what was already said. Add no new claims, checks or opinions. If something looks
  wrong while summarizing, say so in one extra line after "Next" instead of silently fixing it.
- If the answer corrected an earlier claim, the summary uses the corrected version.
- Keep a caveat only if it changes the decision. Drop the rest.
- Plain words. No jargon unless it is the name of the thing.
- Never use the em dash character. Use straight quotes only.
- No headers other than the ones in the shape above. No tables.
